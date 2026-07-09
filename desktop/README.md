# SurrealDB & Spectron for Claude Desktop

Claude Desktop has no plugin/marketplace format, so installation is two manual steps: add the MCP servers, then upload the skills. The skill folders live under [`../plugins/`](../plugins/) — they are shared with the Claude Code / Cowork install path so there is only one source of truth.

Both servers are HTTP and both need **your own instance's `/mcp` URL** — there is no default. Install only the plugin(s) you use.

## 1. Add the MCP servers

### Option A — Connectors UI (recommended)

Settings → Connectors → Add custom connector. Add one entry per server you want:

| Name | URL | Header |
|---|---|---|
| `surrealdb-database` | your SurrealDB `/mcp` endpoint (e.g. `http://127.0.0.1:8000/mcp` or `https://<your-instance>/mcp`) | `Authorization: Bearer <SURREALDB_MCP_TOKEN>` |
| `spectron` | your Spectron `/mcp` endpoint (`https://<your-spectron-instance>/mcp`) | `Authorization: Bearer <SPECTRON_MCP_TOKEN>` |

### Option B — Manual config

Merge [`claude_desktop_config.example.json`](claude_desktop_config.example.json) into your `claude_desktop_config.json` (macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`). Replace every `REPLACE_WITH_*` placeholder — including the URLs — with real values, delete any server block you don't use, then restart Claude Desktop.

## 2. Upload the skills

Settings → Skills → Upload skill. Upload the folders for the plugin(s) you use:

**SurrealDB:**

- [`../plugins/surrealdb/skills/mcp/`](../plugins/surrealdb/skills/mcp/) — when and how to use the Database MCP
- [`../plugins/surrealdb/skills/surql-formatter/`](../plugins/surrealdb/skills/surql-formatter/) — formatting `.surql` files
- [`../plugins/surrealdb/skills/surrealql/`](../plugins/surrealdb/skills/surrealql/) — writing idiomatic SurrealQL
- [`../plugins/surrealdb/skills/surrealdb-vector/`](../plugins/surrealdb/skills/surrealdb-vector/) — vector search and embeddings
- [`../plugins/surrealdb/skills/surrealdb-python/`](../plugins/surrealdb/skills/surrealdb-python/) — Python SDK

**Spectron:**

- [`../plugins/spectron/skills/mcp/`](../plugins/spectron/skills/mcp/) — when and how to use the Spectron MCP

## Auto-formatting `.surql` files

Claude Desktop has no hook system, so the `PostToolUse` formatter hook used on Claude Code is unavailable here. Instead, the `surql-formatter` skill instructs the model to invoke `surqlfmt --write` itself after each `.surql` edit. This is model-driven and best-effort — it relies on the model following the skill, not a deterministic post-tool hook.

If you want stricter enforcement, run the formatter in CI or as a pre-commit hook in your project:

```sh
npx -y --package=@surrealdb/surql-fmt surqlfmt --check path/to/file.surql
```

## Updating

Pull the latest of this repo and re-upload any skill folders whose contents changed. MCP server config does not need to be touched unless URLs or tokens change.
