---
name: surql-formatter
description: Use when the user asks Claude to format, lint, or check SurrealQL files (.surql), or when troubleshooting the auto-format hook bundled with this plugin.
---

# SurQL Formatter

This plugin auto-formats every `.surql` file Claude edits using [`@surrealdb/surql-fmt`](https://www.npmjs.com/package/@surrealdb/surql-fmt). The hook runs after `Edit`, `Write`, and `MultiEdit` tool calls and is configured in `plugins/surrealdb/settings.json`.

## How the hook works

`PostToolUse` matchers on `Edit|Write|MultiEdit` pipe the tool input through `jq`, keep only paths ending in `.surql`, then run:

```sh
npx -y --package=@surrealdb/surql-fmt surqlfmt --write <file>
```

The hook is non-blocking — formatter errors are swallowed and the agent continues. First run downloads the npm package and may take a few seconds.

## Manual usage

Format a file in place:

```sh
npx -y --package=@surrealdb/surql-fmt surqlfmt --write path/to/file.surql
```

Check formatting without writing (CI use — exits 1 on diff):

```sh
npx -y --package=@surrealdb/surql-fmt surqlfmt --check path/to/file.surql
```

Format from stdin:

```sh
echo 'SELECT * FROM user;' | npx -y --package=@surrealdb/surql-fmt surqlfmt --stdin
```

Common flags: `--indent <n>`, `--indent-char space|tab`, `--max-line-length <n>`.

## Limitations

- `.surql` files only. SurQL strings embedded in JS/TS/Python/Rust sources are not touched — there is no language-server or tagged-template walker. To format an embedded query, copy it into a `.surql` scratch file, format it, and paste back.
- Parse errors leave the file unchanged and the hook silently moves on. If a file stops formatting, run `surqlfmt --check` manually to see the error.

## Disabling the hook

Remove the `PostToolUse` block from `plugins/surrealdb/settings.json`, or delete the file. Manual `surqlfmt` invocations continue to work.
