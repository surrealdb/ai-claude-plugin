# SurrealDB Claude Marketplace

A Claude plugin marketplace from SurrealDB. It hosts two plugins:

| Plugin | What it connects | Ships |
|---|---|---|
| [`surrealdb`](plugins/surrealdb/) | Your SurrealDB instance's `/mcp` route | Instance MCP, SurrealQL skills, `.surql` auto-formatter |
| [`spectron`](plugins/spectron/) | Your Spectron instance's `/mcp` route | Instance MCP + a usage skill |

Both plugins connect to an **instance MCP at `<your-instance>/mcp`** — you point them at your own instance; there is no default URL. Supports **Claude Code**, **Cowork**, and **Claude Desktop**.

## Install (Claude Code / Cowork)

Add this repo as a marketplace, then install whichever plugins you want:

```
/plugin marketplace add surrealdb/ai-claude-plugin
/plugin install surrealdb@surrealdb
/plugin install spectron@surrealdb
```

Claude Code reads each plugin directory directly — MCP servers, skills, and (for SurrealDB) the auto-format hook load automatically. Then set the required environment variables below and restart Claude Code.

For **Claude Desktop** (no plugin/marketplace format), install manually — see [`desktop/README.md`](desktop/README.md).

## `surrealdb` plugin

### MCP server (`plugins/surrealdb/.mcp.json`)

| Name | Endpoint | Purpose |
|---|---|---|
| `surrealdb-database` | `${SURREALDB_MCP_URL}` | Data plane: query, schema, records, permissions on your SurrealDB instance |

### Skills (`plugins/surrealdb/skills/`)

- `mcp`: when and how to use the Database MCP (local)
- `surql-formatter`: running `@surrealdb/surql-fmt` and the auto-format hook (local)
- `surrealql`: writing idiomatic SurrealQL (synced from `surrealdb/agent-skills`)
- `surrealdb-vector`: vector search and embeddings (synced)
- `surrealdb-python`: the Python SDK (synced)

### Auto-format hook (`plugins/surrealdb/settings.json`)

A `PostToolUse` hook on `Edit|Write|MultiEdit` runs [`@surrealdb/surql-fmt`](https://www.npmjs.com/package/@surrealdb/surql-fmt) on every `.surql` file Claude touches. Non-blocking: formatter errors don't block the agent.

### Configuration

The Database MCP has **no default URL** — set it to your instance's `/mcp` endpoint plus a bearer token before launching Claude Code:

```sh
export SURREALDB_MCP_URL="https://my-host.example.com/mcp"   # or http://127.0.0.1:8000/mcp for a local instance
export SURREALDB_MCP_TOKEN="<bearer-token>"
```

If `SURREALDB_MCP_URL` is unset, the server won't connect.

## `spectron` plugin

### MCP server (`plugins/spectron/.mcp.json`)

| Name | Endpoint | Purpose |
|---|---|---|
| `spectron` | `${SPECTRON_MCP_URL}` | Connects Claude to your Spectron instance |

### Skills (`plugins/spectron/skills/`)

- `mcp`: when and how to use the Spectron MCP (local)

### Memory hooks (`plugins/spectron/hooks/`)

Automatically recall memories into context and persist them back, using the same `/mcp` endpoint and token as the MCP server (`context_id` is inferred from the token):

| Event | Spectron tool | What it does |
|---|---|---|
| `SessionStart` | `recall` | Injects general user/project memories at session start. |
| `UserPromptSubmit` | `recall` | Injects memories relevant to each prompt. |
| `Stop` | `remember` | Persists the latest exchange (`infer:"full"`). |

All three run [`hooks/spectron-memory.sh`](plugins/spectron/hooks/spectron-memory.sh) (needs `curl` + `jq`) and **fail open** — if Spectron is unset, unreachable, or slow, they exit silently and never block you. Plugin hooks run in **Claude Code / Cowork** only (not Claude Desktop).

### Configuration

The Spectron MCP has **no default URL** — set it to your instance's `/mcp` endpoint plus a bearer token:

```sh
export SPECTRON_MCP_URL="https://your-spectron-instance.example.com/mcp"
export SPECTRON_MCP_TOKEN="<bearer-token>"
```

If `SPECTRON_MCP_URL` is unset, the server won't connect (and the memory hooks stay inert).

Optional memory-hook knobs:

```sh
export SPECTRON_MEMORY_HOOKS=off      # disable the memory hooks (MCP server stays connected)
export SPECTRON_HOOK_TIMEOUT=8        # per-call network timeout in seconds (default 8)
```

## Customizing

**Disable the format hook**: delete the `hooks` block from `plugins/surrealdb/settings.json`.

**Refresh upstream skills** (SurrealDB plugin): pulls latest from `surrealdb/agent-skills`:

```sh
bash scripts/sync-agent-skills.sh
```

The local skills (`mcp`, `surql-formatter`) are protected from being overwritten.
