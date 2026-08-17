---
name: surrealdb-mcp
description: Use when the user asks Claude to query, inspect, administer, or troubleshoot SurrealDB through the bundled MCP server — SurrealDB's managed MCP at mcp.surrealdb.com, covering both data (SurrealQL, schema, records) and account-level work (instances, organizations, spend).
---

# SurrealDB MCP

This plugin wires up one MCP server in `.mcp.json`:

- **`surrealdb`** — SurrealDB's managed MCP server at `https://mcp.surrealdb.com`, scoped to the user's SurrealDB account. It covers both planes: running SurrealQL and inspecting schema against the account's instances, and managing the account itself — deploying and upgrading instances, resizing resources, organizations and access, and spend.

There is **nothing to configure**. The server authenticates over OAuth: the first call opens a browser to sign in with the user's Surreal ID. If a call comes back unauthenticated, tell the user to sign in when prompted — don't go hunting for environment variables or a URL.

## What it's for

| Task | Notes |
|---|---|
| Run a SurrealQL query, inspect tables/fields, define schema, work with records | Requires the target instance on SurrealDB 3.2.3+ |
| Deploy a new instance, upgrade a version, resize resources | Billable — confirm first |
| Invite users to an organization, change access levels | Confirm first |
| List instances, check monthly spend | Read-only, no confirmation needed |

## Local and self-hosted instances

`mcp.surrealdb.com` runs in SurrealDB's infrastructure, so **it cannot reach a `localhost` or private-network instance**. If the user wants Claude talking to a local `surreal start` instance or a self-hosted deployment, that's the separate **`surrealdb-local`** plugin:

```
/plugin install surrealdb-local@surrealdb
```

It adds a `surrealdb-local` MCP server pointed at `${SURREALDB_MCP_URL}` (the instance's own `/mcp` route) with a bearer token. See that plugin's skill for setup. When both plugins are installed, route data work for a local instance to `surrealdb-local` and everything account-level to `surrealdb`.

## Headless / unattended use

OAuth needs a browser. Where there isn't one — CI, a remote box, an unattended agent — the user can mint a personal access token at <https://account.surrealdb.com/tokens> and register the server themselves with an explicit header:

```sh
claude mcp add --transport http surrealdb https://mcp.surrealdb.com \
	--header "Authorization: Bearer <personal-access-token>"
```

The URL is the **bare root** — `https://mcp.surrealdb.com`, with no `/mcp` or `/sse` path. Those 404.

## Safety

Every tool call runs against the user's real database and real account. Before:

- `DELETE` or bulk updates
- `DEFINE` / `REMOVE` on tables, fields, indexes, scopes, accesses
- Changing permissions or auth
- Creating, resizing, upgrading, or deleting an instance — these change what the user is billed
- Inviting users to an organization or changing anyone's access level

…state the intended change and confirm with the user. Read-only work — `SELECT`, `INFO FOR`, listing instances, checking spend — does not need confirmation.

## Starter requests

- "Sign me in to SurrealDB and show me my instances."
- "Show me the schema for the active namespace and database."
- "Run `SELECT * FROM user LIMIT 5;` against my database."
- "Define a `post` table with `title` and `body` fields and a `published` permission."
- "What have I spent on SurrealDB this month?"
