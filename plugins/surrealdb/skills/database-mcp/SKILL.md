---
name: database-mcp
description: Use when the user asks Claude to inspect, query, administer, or troubleshoot SurrealDB through the bundled MCP servers — either the local Database MCP (the user's own SurrealDB server) or the hosted Cloud MCP at app.surrealdb.com.
---

# SurrealDB MCP

This plugin ships **two** MCP servers wired up in `.mcp.json`:

- **`surrealdb-database`** — the data plane. Talks directly to the user's SurrealDB server over its built-in `/mcp` HTTP route. Use it for inspecting schemas, running SurrealQL, reading/writing records, managing permissions, and everything else inside a namespace/database.
- **`surrealdb-cloud`** — the control plane at `https://app.surrealdb.com/mcp`. Use it to list, create, pause, resume, or delete Cloud instances and organizations. (This endpoint is rolling out — if it is not yet reachable, fall back to database-level work or the Surrealist dashboard.)

Do **not** ask the user to run `surreal mcp stdio` — that path is deprecated. The MCP server is the `/mcp` route exposed by the SurrealDB server itself.

## Picking the right server

| Task | Use |
|---|---|
| Run a SurrealQL query, inspect tables/fields, define schema, manage users/scopes, work with records | `surrealdb-database` |
| Spin up a new Cloud instance, pause/resume, list orgs, check instance status | `surrealdb-cloud` |

When in doubt, default to `surrealdb-database`. Cloud is for instance lifecycle, not data.

## Configuring the Database MCP

The Database MCP URL defaults to `http://127.0.0.1:8000/mcp`. To point at a different SurrealDB server, set environment variables before launching Claude Code:

```sh
export SURREALDB_MCP_URL="https://my-host.example.com/mcp"
export SURREALDB_MCP_TOKEN="<bearer-token-or-jwt>"
```

- `SURREALDB_MCP_URL` — full URL including `/mcp` path.
- `SURREALDB_MCP_TOKEN` — bearer token used to authenticate against the SurrealDB server's `/mcp` route.

A SurrealDB `surreal-bearer-...` grant key is **not** an HTTP auth token. If the user only has signin credentials (root user/pass, scope `SIGNIN`/`SIGNUP`), they must exchange those for an access token first.

## Configuring the Cloud MCP

`surrealdb-cloud` authenticates with a SurrealDB Cloud **Personal Access Token (PAT)**. Generate one in the SurrealDB Cloud dashboard (`app.surrealdb.com` → account settings → personal access tokens) and export it before launching Claude Code:

```sh
export SURREALDB_CLOUD_TOKEN="<personal-access-token>"
```

The token is sent as `Authorization: Bearer ${SURREALDB_CLOUD_TOKEN}`. Treat it like an API key: scope it tightly and rotate when leaked.

## Safety

Every MCP query and mutation tool runs against the user's real database. Before:

- `DELETE` or bulk updates
- `DEFINE` / `REMOVE` on tables, fields, indexes, scopes, accesses
- Changing permissions or auth
- Creating, pausing, or deleting Cloud instances

…state the intended change and confirm with the user. Read-only `SELECT` / `INFO FOR` queries do not need confirmation.

## Starter requests

- "Show me the schema for the active namespace and database."
- "Run `SELECT * FROM user LIMIT 5;` against my local database."
- "Define a `post` table with `title` and `body` fields and a `published` permission."
- "List my SurrealDB Cloud instances."
- "Pause the `analytics-staging` Cloud instance."
