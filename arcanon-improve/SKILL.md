---
name: arcanon-improve
description: This skill should be used when the user says "the scanner should detect X", "recommend scanner changes", "file a scanner improvement", "this can't be fixed with a pattern", or when a gap requires changes to the scanner's source code, CDN patterns, or detection engine rather than .arcanon.toml.
---

Analyze gaps in the Arcanon Scanner's detection and recommend concrete changes to its codebase or CDN pattern registry. Write recommendations to a `scanner-improvements.md` file.

## Tools

- **Read** — read scanner source code (`src/plugin/lang/`, `src/patterns/mod.rs`, `src/core/scanner.rs`, `src/libres/mod.rs`, `src/wrapper/mod.rs`)
- **Grep** — search scanner source for existing detection logic
- **Write** — write `scanner-improvements.md` with structured recommendations
- **Bash** — run `cargo test` to verify proposed changes don't break existing tests (if making code changes)

## Scanner architecture

Three layers of detection, from highest to lowest accuracy:

1. **Compiled plugins** (`src/plugin/lang/`) — AST-based, per-language. Detect framework routes and connection calls. Require a scanner release to update.
2. **CDN patterns** (fetched from `patterns.arcanon.dev`) — string matching with import gates. Updated without a scanner release.
3. **User patterns** (`.arcanon.toml`) — same format as CDN patterns but local to the repo.

## Classify the gap

| Gap type | Fix location |
|----------|-------------|
| Missing library detection (string-matchable) | CDN pattern |
| Framework routing not supported | Compiled plugin (`src/plugin/lang/`) |
| AST-level extraction needed | Compiled plugin |
| New target_extraction strategy needed | Pattern engine (`src/patterns/mod.rs`) |
| False positive in existing detection | Plugin or CDN pattern |
| Library resolution producing too many findings | `src/core/scanner.rs` or `src/libres/mod.rs` |

## Write the recommendation

Use Write to create or append to `scanner-improvements.md` in the current directory. Each recommendation follows this format:

```markdown
## [SHORT TITLE]

**Type:** CDN pattern | Plugin change | Engine change
**Scope:** [Who benefits]
**Complexity:** Trivial | Moderate | Significant
**Priority:** High | Medium | Low

### Problem
[What the scanner misses and why — include file paths and line numbers from the scanner source]

### Proposed fix
[Concrete change — CDN pattern JSON, code location and diff, or engine design]

### Verification
[Example code that should be detected after the fix]
```

For **CDN pattern** recommendations, write the pattern in JSON format. For **plugin changes**, use Read to examine the existing plugin code and reference specific file paths and line numbers.

## Rules

- **Patterns first.** If a gap can be solved with a CDN or user pattern, do not recommend a plugin change.
- **Read the scanner source** via Read before recommending plugin changes. Reference specific files and line numbers.
- **Be specific.** "Improve TypeScript detection" is not a recommendation. "Add Hono route detection to `src/plugin/lang/typescript.rs` matching `app.get('/path', handler)`" is.
- **Consider false positives.** Every new detection risks matching non-connections. Include how to avoid false matches.
- **One recommendation per gap.** Do not bundle unrelated changes.
