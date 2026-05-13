---
name: surql-formatter
description: Use after editing or writing any `.surql` file, or when the user asks Claude to format, lint, or check SurrealQL files.
---

# SurQL Formatter

**Primary instruction:** after editing or writing any `.surql` file, run [`@surrealdb/surql-fmt`](https://www.npmjs.com/package/@surrealdb/surql-fmt) on it:

```sh
npx -y --package=@surrealdb/surql-fmt surqlfmt --write path/to/file.surql
```

First run downloads the npm package and may take a few seconds. Formatter errors are non-fatal — if the formatter fails, leave the file as the user edited it and continue.

## Host behavior

- **Claude Code / Cowork**: a `PostToolUse` hook in [`plugins/surrealdb/settings.json`](../../settings.json) already runs `surqlfmt --write` on every `.surql` file after `Edit`, `Write`, or `MultiEdit`. The model does not need to invoke the formatter — it happens automatically.
- **Claude Desktop**: no hook system. The model must invoke `surqlfmt` via Bash after each `.surql` edit, per the primary instruction above.

When unsure which host you are running in, just run the formatter — re-running it on already-formatted output is a no-op.

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
- Parse errors leave the file unchanged. If a file stops formatting, run `surqlfmt --check` manually to see the error.

## Disabling

- **Claude Code / Cowork**: remove the `PostToolUse` block from [`plugins/surrealdb/settings.json`](../../settings.json), or delete the file.
- **Claude Desktop**: remove this skill, or instruct the model not to run the formatter.

Manual `surqlfmt` invocations continue to work either way.
