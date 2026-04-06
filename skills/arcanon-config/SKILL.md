---
name: arcanon-config
description: This skill should be used when the user asks to "configure the scanner", "add a pattern", "update .arcanon.toml", "disable a plugin", "exclude paths", mentions ".arcanon.toml", or reports that the scanner missed a connection from a specific library.
---

## Tools

- **Read** (`read_file`) — read `.arcanon.toml` and source files to understand current config and code patterns
- **Grep** (`grep_search`) — search source files for import statements and function calls
- **Write/Edit** (`write_file`/`replace`) — create or update `.arcanon.toml`
- **Bash** (`run_shell_command`) — run `arcanon --dry-run -vv` to verify patterns work

Configure the Arcanon Scanner by writing `.arcanon.toml` patterns and settings. The file lives at the repo root and is committed to version control.

## Workflow

### When the scanner missed a connection

1. **Read the source code** — use Grep to find the import statement and call site the scanner missed
2. **Determine the import gate** — the import/require string that identifies the library (wrap in an array for the `import_gate` field)
3. **Determine the match** — the function call that creates the connection, with enough context to be unique
4. **Choose target extraction** — `none`, `first_string_arg`, `url_hostname`, or `named_arg:KEY`
5. **Write the pattern** — add a `[[patterns]]` entry to `.arcanon.toml` using Edit or Write
6. **Verify** — run `arcanon --dry-run -vv` via Bash and check the output for new findings

### When disabling or excluding

- Disable a built-in plugin: add to `[scanner.plugins].disabled`
- Disable a CDN pattern by ID: add to `[scanner.patterns].disabled`
- Exclude paths: add globs to `[scanner.exclude].paths`

## Key constraints

- Only write config for implemented features. `[services]` overrides and `[[connections.manual]]` are **not yet implemented** — inform the user if they need these.
- `file_patterns` is parsed but **not enforced** (known bug) — do not rely on it for scoping.
- Always read the actual code before writing patterns. Never guess `import_gate` or `match` values from library names alone.
- One pattern per library. Multiple detections within a pattern are fine.

## Reference

For the full `.arcanon.toml` schema, field details, and target extraction strategies, consult **`references/schema.md`**.
