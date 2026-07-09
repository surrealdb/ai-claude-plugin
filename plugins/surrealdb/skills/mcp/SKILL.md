---
name: mcp
description: Use when the user asks Claude to inspect, query, administer, or troubleshoot SurrealDB through the bundled Database MCP server — the user's own SurrealDB instance, reached over its built-in /mcp HTTP route.
---

# SurrealDB MCP

This plugin wires up one MCP server in `.mcp.json`:

- **`surrealdb-database`** — the data plane. Talks directly to the user's SurrealDB instance over its built-in `/mcp` HTTP route. Use it for inspecting schemas, running SurrealQL, reading/writing records, managing permissions, and everything else inside a namespace/database.

## What it's for

| Task | Use |
|---|---|
| Run a SurrealQL query, inspect tables/fields, define schema, manage users/scopes, work with records | `surrealdb-database` |

## Configuring the Database MCP

The Database MCP URL is **required** — there is no default. Set it to your instance's `/mcp` endpoint, along with a bearer token, before launching Claude Code:

```sh
export SURREALDB_MCP_URL="https://my-host.example.com/mcp"
export SURREALDB_MCP_TOKEN="<bearer-token-or-jwt>"
```

- `SURREALDB_MCP_URL` — full URL including the `/mcp` path (e.g. `http://127.0.0.1:8000/mcp` for a local instance, or `https://<your-instance>/mcp` for a remote one).
- `SURREALDB_MCP_TOKEN` — bearer token used to authenticate against the SurrealDB instance's `/mcp` route.

If `SURREALDB_MCP_URL` is unset, the server won't connect — tell the user to export it and restart Claude Code.

A SurrealDB `surreal-bearer-...` grant key is **not** an HTTP auth token. If the user only has signin credentials (root user/pass, scope `SIGNIN`/`SIGNUP`), they must exchange those for an access token first.

## Safety

Every MCP query and mutation tool runs against the user's real database. Before:

- `DELETE` or bulk updates
- `DEFINE` / `REMOVE` on tables, fields, indexes, scopes, accesses
- Changing permissions or auth

…state the intended change and confirm with the user. Read-only `SELECT` / `INFO FOR` queries do not need confirmation.

## Starter requests

- "Show me the schema for the active namespace and database."
- "Run `SELECT * FROM user LIMIT 5;` against my database."
- "Define a `post` table with `title` and `body` fields and a `published` permission."
