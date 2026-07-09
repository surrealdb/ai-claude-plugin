---
name: mcp
description: Use when the user references Spectron, or when Spectron tools appear in the active toolset — this plugin connects Claude to a Spectron instance over its /mcp HTTP route.
---

# Spectron MCP

This plugin wires up one MCP server in `.mcp.json`:

- **`spectron`** — connects to the user's Spectron instance over its `/mcp` HTTP route.

Spectron is **not** a SurrealDB database server. Use it only when the user explicitly references Spectron or when its tools appear in the active toolset. Don't route SurrealQL, schema, or record work here — that belongs to the SurrealDB plugin's `surrealdb-database` server.

## Configuring the Spectron MCP

The Spectron MCP URL is **required** — there is no default. Set it to your instance's `/mcp` endpoint, along with a bearer token, before launching Claude Code:

```sh
export SPECTRON_MCP_URL="https://your-spectron-instance.example.com/mcp"
export SPECTRON_MCP_TOKEN="<bearer-token>"
```

- `SPECTRON_MCP_URL` — full URL including the `/mcp` path.
- `SPECTRON_MCP_TOKEN` — bearer token used to authenticate against the Spectron instance's `/mcp` route.

If `SPECTRON_MCP_URL` is unset, the server won't connect — tell the user to export it and restart Claude Code.

## Using the tools

The Spectron tool surface is whatever the connected instance exposes — do not assume specific tools exist. Check the active toolset before invoking, and prefer read-only operations unless the user has asked for a change. For any mutating action, state the intended change and confirm with the user first.
