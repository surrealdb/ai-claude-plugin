---
name: surrealdb-local
description: Use when the user wants Claude to query or inspect a local, self-hosted, or air-gapped SurrealDB instance over its built-in /mcp HTTP route — including setting up SURREALDB_MCP_URL and a scoped token, or troubleshooting why the local MCP server won't connect.
---

# SurrealDB (local / self-hosted)

This plugin wires up one MCP server in `.mcp.json`:

- **`surrealdb-local`** — the data plane for an instance the user runs themselves. Talks to that instance over SurrealDB's built-in `/mcp` HTTP route: SurrealQL, schema, records, permissions.

Use it when the target instance is local (`surreal start` on the developer's machine), self-hosted, or air-gapped. The managed `surrealdb` plugin's server runs in SurrealDB's infrastructure and **cannot reach `localhost` or a private network** — that's the entire reason this plugin exists. Account-level work (deploying Cloud instances, organizations, spend) still belongs to the managed server.

## Configuration

Both variables are **required** — there is no default URL. Set them before launching Claude Code:

```sh
export SURREALDB_MCP_URL="http://127.0.0.1:8000/mcp"   # or https://<your-instance>/mcp
export SURREALDB_MCP_TOKEN="<bearer-token-or-jwt>"
```

- `SURREALDB_MCP_URL` — full URL **including** the `/mcp` path. This is the instance's own route, unlike the managed server, which is served from its bare root.
- `SURREALDB_MCP_TOKEN` — bearer token authenticating against that route.

If `SURREALDB_MCP_URL` is unset, the server won't connect — tell the user to export it and restart Claude Code.

## Getting a token

**Use a scoped database user, not root.** The instance resolves the request's `Authorization` header into a real session with a per-request subject check, so the token's permissions are the agent's permissions. Define a user with only the access the work needs and sign in as that user to get a token:

```surql
DEFINE USER claude ON DATABASE PASSWORD "…" ROLES EDITOR;
```

Then exchange those credentials for an access token via the instance's `/signin` route and use the returned JWT as `SURREALDB_MCP_TOKEN`.

A SurrealDB `surreal-bearer-...` grant key is **not** an HTTP auth token. If the user only has signin credentials (root user/pass, scope `SIGNIN`/`SIGNUP`), they must exchange those for an access token first.

## Two things to get right about the server

- The `/mcp` route is **capability-gated**. An operator can disable it with `--deny-http mcp`; if the route 404s on an instance that is otherwise up, check that first.
- Don't reach for `surreal mcp` (the stdio CLI). It does **not** attach to the running instance — it starts its own embedded datastore (in-memory by default), so the agent sees a different, empty database, and it can't be pointed at a running instance's data directory because RocksDB and SurrealKV hold an exclusive lock. It also grants owner-level access on every tool call. It's an operator tool; the HTTP route above is the right path for an agent.

## Safety

Every query and mutation runs against the user's real database. Before `DELETE` or bulk updates, `DEFINE`/`REMOVE` on tables, fields, indexes, scopes or accesses, or any change to permissions or auth — state the intended change and confirm. Read-only `SELECT` / `INFO FOR` queries do not need confirmation.

## Starter requests

- "Connect to my local SurrealDB and show me the schema."
- "Run `SELECT * FROM user LIMIT 5;` against my dev instance."
- "Why can't Claude see my tables?" — check `SURREALDB_MCP_URL` points at the running instance, not an embedded `surreal mcp` datastore.
