# SurrealDB Claude Plugin

Connects Claude to SurrealDB. Ships two MCP servers, five skills, and an auto-formatter for SurrealQL.

## What you get

### MCP servers (`plugins/surrealdb/.mcp.json`)

| Name | Endpoint | Purpose |
|---|---|---|
| `surrealdb-database` | `${SURREALDB_MCP_URL:-http://127.0.0.1:8000/mcp}` | Data plane: query, schema, records, permissions on the user's SurrealDB server |
| `surrealdb-cloud` | `https://app.surrealdb.com/mcp` | Control plane: create/list/pause/resume SurrealDB Cloud instances (rolling out) |

Both speak MCP over HTTP via the `/mcp` route built into SurrealDB. There is no separate `surrealmcp` binary.

### Skills (`plugins/surrealdb/skills/`)

- `database-mcp`: when and how to use the two MCP servers above (local)
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

### Cloud MCP

Authenticates with a SurrealDB Cloud Personal Access Token. Generate one in the Cloud dashboard at `app.surrealdb.com` (account settings) and export it:

```sh
export SURREALDB_CLOUD_TOKEN="<personal-access-token>"
```

## Customizing

**Disable the format hook**: delete the `hooks` block from `plugins/surrealdb/settings.json`.

**Refresh upstream skills**: pulls latest from `surrealdb/agent-skills`:

```sh
bash scripts/sync-agent-skills.sh
```

The local skills (`database-mcp`, `surql-formatter`) are protected from being overwritten.
