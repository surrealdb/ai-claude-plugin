---
name: mcp
description: Use when the user asks Claude to inspect, query, administer, or troubleshoot SurrealDB through the bundled MCP servers — the local Database MCP (the user's own SurrealDB server), the hosted Cloud MCP at app.surrealdb.com, or Spectron.
---

# SurrealDB MCP

This plugin wires up MCP servers in `.mcp.json`:

- **`surrealdb-database`** — the data plane. Talks directly to the user's SurrealDB server over its built-in `/mcp` HTTP route. Use it for inspecting schemas, running SurrealQL, reading/writing records, managing permissions, and everything else inside a namespace/database.
- **`surrealdb-cloud`** — the control plane at `https://app.surrealdb.com/mcp`. Optional: only active when the user has set `SURREALDB_CLOUD_TOKEN`. Use it to list, create, pause, resume, or delete Cloud instances and organizations. If the endpoint is unreachable or the user has no Cloud account, fall back to database-level work or the Surrealist dashboard.
- **`spectron`** — defaults to `https://spectron.surrealdb.com/mcp` (override with `SPECTRON_MCP_URL`); requires `SPECTRON_MCP_TOKEN` for authenticated calls. Not a SurrealDB server — use it only when the user explicitly references Spectron or its tools appear in the active toolset.

## Picking the right server

| Task | Use |
|---|---|
| Run a SurrealQL query, inspect tables/fields, define schema, manage users/scopes, work with records | `surrealdb-database` |
| Spin up a new Cloud instance, pause/resume, list orgs, check instance status | `surrealdb-cloud` |
| Anything explicitly Spectron-related, or invoking a tool that the `spectron` server exposes | `spectron` |

When in doubt, default to `surrealdb-database` — it's the only required server. Cloud is opt-in (instance lifecycle only, not data) and Spectron is orthogonal to the SurrealDB servers — don't route SurrealQL or Cloud lifecycle work to it.

## Configuring the Database MCP

The Database MCP URL defaults to `http://127.0.0.1:8000/mcp`. To point at a different SurrealDB server, set environment variables before launching Claude Code:

```sh
export SURREALDB_MCP_URL="https://my-host.example.com/mcp"
export SURREALDB_MCP_TOKEN="<bearer-token-or-jwt>"
```

- `SURREALDB_MCP_URL` — full URL including `/mcp` path.
- `SURREALDB_MCP_TOKEN` — bearer token used to authenticate against the SurrealDB server's `/mcp` route.

A SurrealDB `surreal-bearer-...` grant key is **not** an HTTP auth token. If the user only has signin credentials (root user/pass, scope `SIGNIN`/`SIGNUP`), they must exchange those for an access token first.

## Configuring the Cloud MCP (optional)

Only needed if the user has a SurrealDB Cloud account. `surrealdb-cloud` authenticates with a **Personal Access Token (PAT)** — generate one in the SurrealDB Cloud dashboard (`app.surrealdb.com` → account settings → personal access tokens) and export it before launching Claude Code:

```sh
export SURREALDB_CLOUD_TOKEN="<personal-access-token>"
```

The token is sent as `Authorization: Bearer ${SURREALDB_CLOUD_TOKEN}`. Treat it like an API key: scope it tightly and rotate when leaked. If `SURREALDB_CLOUD_TOKEN` is unset, do not attempt Cloud operations — tell the user they need a Cloud PAT.

## Configuring the Spectron MCP

Defaults to `https://spectron.surrealdb.com/mcp`. Override only to point at a self-hosted Spectron:

```sh
export SPECTRON_MCP_URL="https://your-spectron-host.example.com/mcp"   # optional override
export SPECTRON_MCP_TOKEN="<bearer-token>"
```

`SPECTRON_MCP_TOKEN` is required for authenticated calls. The Spectron tool surface is whatever the connected Spectron instance exposes — do not assume specific tools exist; check the active toolset before invoking.

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
