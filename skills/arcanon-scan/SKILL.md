---
name: arcanon-scan
description: This skill should be used when the user asks to "scan my repo", "run the scanner", "what did the scanner miss", "improve scan accuracy", "find missing connections", or wants to analyze scan results and add patterns for gaps.
---

## Tools

- **Bash** (`run_shell_command`) — run `arcanon --dry-run -vv`, check for logs in `~/.arcanon/logs/`
- **Read** (`read_file`) — read scan output JSON, source files, manifests for dependencies
- **Grep** (`grep_search`) — search source files for import statements and connection calls not in scan results
- **Glob** (`glob`) — find manifest files and source files by pattern
- **Write/Edit** (`write_file`/`replace`) — create or update `.arcanon.toml` with new patterns

Run the Arcanon Scanner, compare results against source code, identify missed connections, and write `.arcanon.toml` patterns to close the gaps.

## Workflow

### Step 1: Check for existing scan

Use Bash to check for recent scan logs:
```bash
ls -lt ~/.arcanon/logs/ 2>/dev/null | head -5
```

If a recent scan exists, ask the user whether to use those results or run a fresh scan.

### Step 2: Run the scanner

Execute via Bash with output redirection for reliable JSON parsing:
```bash
arcanon --dry-run -vv --output results.json 2>scan.log
```

If the scanner is not installed, inform the user:
```bash
curl -fsSL https://arcanon.dev/install.sh | sh
```

### Step 3: Summarize what was detected

Use Read to parse the `results.json` file. Extract and present:
- Services found (names, root paths)
- Endpoints found (method, path, framework)
- Connections found (source, target, protocol, pattern_id if any)

Check for pattern IDs in the connection data to identify which findings come from custom patterns versus built-in plugins.

### Step 4: Find gaps

Use Glob to find manifest files (`package.json`, `requirements.txt`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `Gemfile`, `pom.xml`, `build.gradle`, `.csproj`). Use Read to check installed dependencies.

Use Grep to search for import statements and client calls from those libraries. Cross-reference against the scan's connection findings.

A gap is: a library that is imported and called in the code but does not appear in the scan results.

**Only flag real gaps.** The scanner already detects 90+ libraries. Read the scan output carefully before claiming something is missing.

### Step 5: Write patterns

For each gap, use Read and Grep to find:
1. The exact import statement (wrap in an array for `import_gate`)
2. The exact function call (becomes `match`)
3. Whether the call takes a URL/string/named argument (determines `target_extraction`)

Use Edit to add `[[patterns]]` entries to `.arcanon.toml`. Refer to `references/schema.md` for the full pattern schema.

### Step 6: Verify

Run the scanner again via Bash:
```bash
arcanon --dry-run -vv --output results-after.json 2>scan-after.log
```

Compare results using Read. Report:
- New connections detected
- Patterns that didn't produce results (investigate verbose logs in `scan-after.log` to fix or remove them)
- Remaining gaps that patterns cannot solve (inform the user these require scanner improvements)

## Rules

- Always read actual code via Read/Grep before writing patterns. Never guess.
- Do not duplicate what the scanner already detects. Read scan output first.
- Be conservative — high-confidence patterns that catch 80% of calls are better than low-confidence patterns with false positives.
- If a pattern fails verification, use the verbose logs to diagnose the issue before removing it.
- `[services]` and `[[connections.manual]]` are not yet implemented — note gaps that need these as limitations.
