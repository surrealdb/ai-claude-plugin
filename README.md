# SurrealDB Claude Plugin

Connects Claude to SurrealDB. Ships three MCP servers, five skills, and an auto-formatter for SurrealQL. Supports **Claude Code**, **Cowork**, and **Claude Desktop**.

## Install

### Claude Code

Install via the plugin marketplace (or point Claude Code at this repo). Claude Code reads [`plugins/surrealdb/`](plugins/surrealdb/) directly — MCP servers, skills, and the auto-format hook all load automatically.

### Cowork

Same plugin layout as Claude Code — install from this repo in a Cowork session. The `.claude-plugin/` manifest works as-is.

### Claude Desktop

Claude Desktop has no plugin/marketplace format, so installation is manual: add the two MCP servers as Connectors and upload the skill folders via the Skills UI. Step-by-step in [`desktop/README.md`](desktop/README.md). The auto-format hook is unavailable on Desktop — the `surql-formatter` skill instructs the model to run the formatter itself after each `.surql` edit (model-driven, not deterministic).

## What you get

### MCP servers (`plugins/surrealdb/.mcp.json`)

| Name | Endpoint | Purpose |
|---|---|---|
| `surrealdb-database` | `${SURREALDB_MCP_URL:-http://127.0.0.1:8000/mcp}` | Data plane: query, schema, records, permissions on the user's SurrealDB server |
| `surrealdb-cloud` | `https://app.surrealdb.com/mcp` | Control plane: create/list/pause/resume SurrealDB Cloud instances. Optional — only used if you have a Cloud account |
| `spectron` | `${SPECTRON_MCP_URL:-https://spectron.surrealdb.com/mcp}` | Spectron MCP |

### Skills (`plugins/surrealdb/skills/`)

- `mcp`: when and how to use the MCP servers above (local)
- `surql-formatter`: running `@surrealdb/surql-fmt` and the auto-format hook (local)
- `surrealql`: writing idiomatic SurrealQL (synced from `surrealdb/agent-skills`)
- `surrealdb-vector`: vector search and embeddings (synced)
- `surrealdb-python`: the Python SDK (synced)

### Auto-format hook (`plugins/surrealdb/settings.json`)

A `PostToolUse` hook on `Edit|Write|MultiEdit` runs [`@surrealdb/surql-fmt`](https://www.npmjs.com/package/@surrealdb/surql-fmt) on every `.surql` file Claude touches. Non-blocking: formatter errors don't block the agent.

## Configuration

### Database MCP

Defaults to a local SurrealDB on `http://127.0.0.1:8000/mcp`. Override with env vars before launching Claude Code:

```sh
export SURREALDB_MCP_URL="https://my-host.example.com/mcp"
export SURREALDB_MCP_TOKEN="<bearer-token>"
```

### Cloud MCP (optional)

Only needed if you use SurrealDB Cloud. Authenticates with a Personal Access Token — generate one in the Cloud dashboard at `app.surrealdb.com` (account settings) and export it:

```sh
export SURREALDB_CLOUD_TOKEN="<personal-access-token>"
```

If `SURREALDB_CLOUD_TOKEN` is unset, the connector loads but authenticated calls fail — leave it unset (or remove the block) if you don't have a Cloud account.

### Spectron MCP

Defaults to `https://spectron.surrealdb.com/mcp`. Override to point at a self-hosted Spectron, and supply a bearer token:

```sh
export SPECTRON_MCP_URL="https://your-spectron-host.example.com/mcp"   # optional override
export SPECTRON_MCP_TOKEN="<bearer-token>"
```

If `SPECTRON_MCP_TOKEN` is unset, the connector still loads but authenticated calls will fail - set the token (or remove the `spectron` block) if you don't intend to use it.

## Customizing

**Disable the format hook**: delete the `hooks` block from `plugins/surrealdb/settings.json`.

**Refresh upstream skills**: pulls latest from `surrealdb/agent-skills`:

```sh
bash scripts/sync-agent-skills.sh
```

The local skills (`mcp`, `surql-formatter`) are protected from being overwritten.
