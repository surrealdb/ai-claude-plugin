# SurrealDB & Agent Memory for Claude Desktop

Claude Desktop has no plugin/marketplace format, so installation is two manual steps: add the MCP server, then upload the skills. The skill folders live under [`../plugins/`](../plugins/) — they are shared with the Claude Code / Cowork install path so there is only one source of truth.

The managed server needs **no configuration** — no URL to find, no token to mint. Add it, sign in with your Surreal ID when prompted, done. Only add the local server if you develop against an instance you run yourself.

> SurrealDB and Agent Memory share one endpoint, so on Desktop you add **one** connector for both — there's no reason to add `https://mcp.surrealdb.com` twice under two names. What Desktop can't do is the ambient memory: the `recall`/`remember` hooks need a hook system, so memory here is on-demand through the Agent Memory tools.

## 1. Add the MCP server

### Option A — Connectors UI (recommended)

Settings → Connectors → Add custom connector:

| Name | URL | Auth |
|---|---|---|
| `surrealdb` | `https://mcp.surrealdb.com` | OAuth — sign in with your Surreal ID when prompted |
| `surrealdb-local` *(optional)* | your instance's `/mcp` endpoint (e.g. `http://127.0.0.1:8000/mcp`) | Header: `Authorization: Bearer <SURREALDB_MCP_TOKEN>` |

The managed URL is the **bare root** — no `/mcp` or `/sse` path; those 404. It carries the SurrealDB *and* Agent Memory tools. Your own instance is the opposite: its route *is* `/mcp`.

### Option B — Manual config

Merge [`claude_desktop_config.example.json`](claude_desktop_config.example.json) into your `claude_desktop_config.json` (macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`). The `surrealdb` block works as-is. For `surrealdb-local`, replace the `REPLACE_WITH_*` placeholders — or delete that block if you don't run your own instance — then restart Claude Desktop.

**For the local server, use a scoped database user rather than root.** The instance turns the request's `Authorization` header into a real session, so the token's permissions are the agent's permissions:

```surql
DEFINE USER claude ON DATABASE PASSWORD "…" ROLES EDITOR;
```

Sign in as that user to get a token. Note that `surreal mcp` (the stdio CLI) is not the right transport here — it starts its own embedded datastore instead of attaching to your running instance, so the agent would see an empty database.

## 2. Upload the skills

Settings → Skills → Upload skill:

- [`../plugins/surrealdb/skills/surrealdb-mcp/`](../plugins/surrealdb/skills/surrealdb-mcp/) — when and how to use the MCP server
- [`../plugins/surrealdb/skills/surql-formatter/`](../plugins/surrealdb/skills/surql-formatter/) — formatting `.surql` files
- [`../plugins/surrealdb/skills/surrealql/`](../plugins/surrealdb/skills/surrealql/) — writing idiomatic SurrealQL
- [`../plugins/surrealdb/skills/surrealdb-vector/`](../plugins/surrealdb/skills/surrealdb-vector/) — vector search and embeddings
- [`../plugins/surrealdb/skills/surrealdb-python/`](../plugins/surrealdb/skills/surrealdb-python/) — Python SDK

If you added the local server, also upload [`../plugins/surrealdb-local/skills/surrealdb-local/`](../plugins/surrealdb-local/skills/surrealdb-local/). For Agent Memory, upload [`../plugins/agent-memory/skills/agent-memory/`](../plugins/agent-memory/skills/agent-memory/) — note its hook sections don't apply on Desktop.

## Auto-formatting `.surql` files

Claude Desktop has no hook system, so the `PostToolUse` formatter hook used on Claude Code is unavailable here. Instead, the `surql-formatter` skill instructs the model to invoke `surqlfmt --write` itself after each `.surql` edit. This is model-driven and best-effort — it relies on the model following the skill, not a deterministic post-tool hook.

If you want stricter enforcement, run the formatter in CI or as a pre-commit hook in your project:

```sh
npx -y --package=@surrealdb/surql-fmt surqlfmt --check path/to/file.surql
```

## Updating

Pull the latest of this repo and re-upload any skill folders whose contents changed. MCP server config does not need to be touched unless URLs or tokens change.
