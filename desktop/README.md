# SurrealDB for Claude Desktop

Claude Desktop has no plugin/marketplace format, so installation is two manual steps: add the MCP servers, then upload the skills. The skill folders live in [`../plugins/surrealdb/skills/`](../plugins/surrealdb/skills/) — they are shared with the Claude Code / Cowork install path so there is only one source of truth.

## 1. Add the MCP servers

Both MCP servers are HTTP. Two options:

### Option A — Connectors UI (recommended)

Settings → Connectors → Add custom connector. Add one entry per server:

| Name | URL | Header |
|---|---|---|
| `surrealdb-database` | `http://127.0.0.1:8000/mcp` (or your remote SurrealDB `/mcp` endpoint) | `Authorization: Bearer <SURREALDB_MCP_TOKEN>` |
| `surrealdb-cloud` | `https://app.surrealdb.com/mcp` | `Authorization: Bearer <SURREALDB_CLOUD_TOKEN>` |

Get a SurrealDB Cloud Personal Access Token from `app.surrealdb.com` (account settings).

### Option B — Manual config

Merge [`claude_desktop_config.example.json`](claude_desktop_config.example.json) into your `claude_desktop_config.json` (macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`). Replace the `REPLACE_WITH_*` placeholders with real tokens. Restart Claude Desktop.

## 2. Upload the skills

Settings → Skills → Upload skill. Upload each of these folders from this repo:

- [`../plugins/surrealdb/skills/database-mcp/`](../plugins/surrealdb/skills/database-mcp/) — when and how to use the two MCP servers
- [`../plugins/surrealdb/skills/surql-formatter/`](../plugins/surrealdb/skills/surql-formatter/) — formatting `.surql` files
- [`../plugins/surrealdb/skills/surrealql/`](../plugins/surrealdb/skills/surrealql/) — writing idiomatic SurrealQL
- [`../plugins/surrealdb/skills/surrealdb-vector/`](../plugins/surrealdb/skills/surrealdb-vector/) — vector search and embeddings
- [`../plugins/surrealdb/skills/surrealdb-python/`](../plugins/surrealdb/skills/surrealdb-python/) — Python SDK

## Auto-formatting `.surql` files

Claude Desktop has no hook system, so the `PostToolUse` formatter hook used on Claude Code is unavailable here. Instead, the `surql-formatter` skill instructs the model to invoke `surqlfmt --write` itself after each `.surql` edit. This is model-driven and best-effort — it relies on the model following the skill, not a deterministic post-tool hook.

If you want stricter enforcement, run the formatter in CI or as a pre-commit hook in your project:

```sh
npx -y --package=@surrealdb/surql-fmt surqlfmt --check path/to/file.surql
```

## Updating

Pull the latest of this repo and re-upload any skill folders whose contents changed. MCP server config does not need to be touched unless URLs or tokens change.
