# SurrealDB Claude Marketplace

A Claude plugin marketplace from SurrealDB. It hosts three plugins:

| Plugin | What it connects | Ships | Setup |
|---|---|---|---|
| [`surrealdb`](plugins/surrealdb/) | SurrealDB's managed MCP at `https://mcp.surrealdb.com` | MCP server, SurrealQL skills, `.surql` auto-formatter | None — sign in with your Surreal ID |
| [`agent-memory`](plugins/agent-memory/) | The same managed MCP at `https://mcp.surrealdb.com` | MCP server, `recall`/`remember` lifecycle hooks + a usage skill | None for the MCP; two env vars for the hooks |
| [`surrealdb-local`](plugins/surrealdb-local/) | Your own instance's `/mcp` route | MCP server + a setup skill | Two env vars |

Install `surrealdb` or `agent-memory` and you're done — no URL to find, no token to mint, just a browser sign-in on first use. Add `surrealdb-local` if you develop against a local or self-hosted instance. Supports **Claude Code**, **Cowork**, and **Claude Desktop** (hooks are Claude Code / Cowork only).

## Install (Claude Code / Cowork)

```
/plugin marketplace add surrealdb/ai-claude-plugin
/plugin install surrealdb@surrealdb
/plugin install agent-memory@surrealdb
```

That's the whole install for both. On its first tool call each MCP server opens a browser to sign in with your Surreal ID; from then on Claude can query your databases, manage your instances, organizations and spend, and work with Agent Memory.

Both plugins point at the same endpoint, so installing both surfaces the same tools twice under two server names. Harmless — install just one if you'd rather not see the duplicates.

### Optional configuration

Two things take environment variables — set them in your shell profile (`~/.zshrc`, `~/.bashrc`) or wherever you launch Claude Code from, then restart Claude Code.

```sh
# surrealdb-local plugin — required, or the plugin shows as failed / "needs attention"
export SURREALDB_MCP_URL="http://127.0.0.1:8000/mcp"   # or https://<your-instance>/mcp
export SURREALDB_MCP_TOKEN="<bearer-token>"

# Agent Memory hooks — optional; without them the MCP server still works,
# memory is just on-demand rather than ambient
export AGENT_MEMORY_MCP_URL="https://your-agent-memory-instance.example.com/mcp"
export AGENT_MEMORY_MCP_TOKEN="<bearer-token>"
```

```
/plugin install surrealdb-local@surrealdb
```

For **Claude Desktop** (no plugin/marketplace format), install manually — see [`desktop/README.md`](desktop/README.md).

## `surrealdb` plugin

### MCP server (`plugins/surrealdb/.mcp.json`)

| Name | Endpoint | Purpose |
|---|---|---|
| `surrealdb` | `https://mcp.surrealdb.com` | Data plane (SurrealQL, schema, records) **and** control plane (instances, organizations, access, spend) for your SurrealDB account |

The endpoint is the **bare root** — `https://mcp.surrealdb.com`, no `/mcp` or `/sse` path; those 404. Auth is OAuth (dynamic client registration, PKCE S256, refresh tokens), so there is nothing to configure and no token in your environment.

**Headless or unattended?** OAuth needs a browser. Mint a personal access token at [account.surrealdb.com/tokens](https://account.surrealdb.com/tokens) and register the server yourself instead:

```sh
claude mcp add --transport http surrealdb https://mcp.surrealdb.com --header "Authorization: Bearer <token>"
```

### Skills (`plugins/surrealdb/skills/`)

- `surrealdb-mcp`: when and how to use the MCP server (local)
- `surql-formatter`: running `@surrealdb/surql-fmt` and the auto-format hook (local)
- `surrealql`: writing idiomatic SurrealQL (synced from `surrealdb/agent-skills`)
- `surrealdb-vector`: vector search and embeddings (synced)
- `surrealdb-python`: the Python SDK (synced)

### Auto-format hook (`plugins/surrealdb/settings.json`)

A `PostToolUse` hook on `Edit|Write|MultiEdit` runs [`@surrealdb/surql-fmt`](https://www.npmjs.com/package/@surrealdb/surql-fmt) on every `.surql` file Claude touches. Non-blocking: formatter errors don't block the agent.

## `surrealdb-local` plugin

`mcp.surrealdb.com` runs in SurrealDB's infrastructure, so it **cannot reach `localhost` or a private network**. This plugin covers local development, self-hosted deployments, and air-gapped installs by talking to the instance's own built-in `/mcp` route.

### MCP server (`plugins/surrealdb-local/.mcp.json`)

| Name | Endpoint | Purpose |
|---|---|---|
| `surrealdb-local` | `${SURREALDB_MCP_URL}` | Data plane: query, schema, records, permissions on an instance you run |

```sh
export SURREALDB_MCP_URL="http://127.0.0.1:8000/mcp"   # full URL, including the /mcp path
export SURREALDB_MCP_TOKEN="<bearer-token>"
```

Two things worth knowing:

- **Use a scoped database user, not root.** The instance resolves the request's `Authorization` header into a real session with a per-request subject check, so the token's permissions are the agent's permissions. `DEFINE USER claude ON DATABASE PASSWORD "…" ROLES EDITOR;`, then sign in as that user for a token.
- **The `/mcp` route is capability-gated** — operators can disable it with `--deny-http mcp`. If the route 404s on a healthy instance, check that first.

Don't use `surreal mcp` (the stdio CLI) for this. It doesn't attach to your running instance — it spins up its own embedded datastore (in-memory by default), so the agent sees a different, empty database, and it can't be pointed at a running instance's data directory because RocksDB and SurrealKV hold an exclusive lock. It's an operator tool, not an agent transport.

### Skills (`plugins/surrealdb-local/skills/`)

- `surrealdb-local`: configuring the local MCP server and minting a scoped token

## `agent-memory` plugin

### MCP server (`plugins/agent-memory/.mcp.json`)

| Name | Endpoint | Purpose |
|---|---|---|
| `agent-memory` | `https://mcp.surrealdb.com` | Agent Memory and context management, on demand |

Same endpoint and same zero-config OAuth as the `surrealdb` plugin — bare root, browser sign-in on first use, nothing in your environment.

### Memory hooks (`plugins/agent-memory/hooks/`)

The hooks are what an MCP server alone can't give you: memory that fires without the model choosing to call a tool.

| Event | Agent Memory tool | What it does |
|---|---|---|
| `SessionStart` | `recall` | Injects general user/project memories at session start. |
| `UserPromptSubmit` | `recall` | Injects memories relevant to each prompt. |
| `Stop` | `remember` | Persists the latest exchange (`infer:"full"`). |

All three run [`hooks/agent-memory.sh`](plugins/agent-memory/hooks/agent-memory.sh) (needs `curl` + `jq`) as a single stateless JSON-RPC round-trip — `context_id` is inferred from the token. `UserPromptSubmit` is a blocking hook on a 12-second timeout, which is why the memory endpoint stays token-authenticated rather than moving to OAuth: one round-trip per prompt instead of three, and a hook process has no access to the MCP client's credential store anyway. So a fully configured user carries two credentials today — OAuth for tools, an Agent Memory token for memory. Collapsing that to one sign-in is roadmap.

```sh
export AGENT_MEMORY_MCP_URL="https://your-agent-memory-instance.example.com/mcp"
export AGENT_MEMORY_MCP_TOKEN="<bearer-token>"
```

The quickest way to get both values: run **`agent-memory mcp`** against your instance — it prints the URL and `Authorization: Bearer <token>`. On a brand-new instance, **`spectrond bootstrap`** mints the first API key and prints the Context id. For a shared/hosted instance, ask whoever operates it.

The hooks **fail open** — if Agent Memory is unset, unreachable, or slow, they exit silently and never block you. **A default install with no token configured transmits nothing**; the MCP server still works, memory is just on-demand rather than ambient. Once configured, the `Stop` hook sends the latest user↔assistant exchange (conversation content) to your Agent Memory endpoint to be stored, and `recall` sends the current prompt to retrieve relevant memories.

Optional knobs:

```sh
export AGENT_MEMORY_HOOKS=off         # disable the memory hooks
export AGENT_MEMORY_HOOK_TIMEOUT=8    # per-call network timeout in seconds (default 8)
```

Plugin hooks run in **Claude Code / Cowork** only (not Claude Desktop).

## Customizing

**Disable the format hook**: delete the `hooks` block from `plugins/surrealdb/settings.json`.

**Refresh upstream skills** (SurrealDB plugin): pulls latest from `surrealdb/agent-skills`:

```sh
bash scripts/sync-agent-skills.sh
```

The local skills (`surrealdb-mcp`, `surql-formatter`) are protected from being overwritten.
