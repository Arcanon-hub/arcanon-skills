# .arcanon.toml Schema Reference

## Implemented Sections

Only the following sections are parsed by the scanner:

- `[scanner]` — hub_url, project_slug
- `[scanner.exclude]` — additional path exclusions
- `[scanner.plugins]` — disable built-in plugins
- `[scanner.patterns]` — disable CDN patterns by ID
- `[[patterns]]` — custom detection patterns with detections

**Not yet implemented (do not write):** `[services]` overrides, `[[connections.manual]]`

## Full Schema

```toml
[scanner]
hub_url = "https://api.arcanon.dev"
project_slug = "my-project"

[scanner.exclude]
paths = ["vendor/**", "legacy/**"]

[scanner.plugins]
disabled = ["ruby", "asyncapi"]

[scanner.patterns]
disabled = ["ts-axios", "py-boto3-sqs"]

[[patterns]]
id = ""                                 # Unique ID (overrides CDN pattern with same ID)
name = ""                               # Display name
description = ""                        # What this detects
languages = []                          # ["typescript", "python", "go", "java", "csharp", "rust", "ruby"]
file_patterns = []                      # ["**/*.ts"] — NOTE: parsed but not enforced yet (bug)
import_gate = []                        # Strings that must appear in the file (OR'd)

[[patterns.detections]]
match = ""                              # String that must appear on the line
kind = "connection"                     # Always "connection"
protocol = ""                           # Free string: rest, grpc, kafka, redis, postgres, etc.
confidence = ""                         # "high", "medium", or "low"
target_extraction = ""                  # How to extract the target (see below)
```

## Pattern Field Details

### import_gate

An array of strings searched as a substring match against entire file content. Acts as a cheap pre-filter — if the file doesn't contain any of these strings, the pattern is skipped entirely. Multiple entries are OR'd.

Good gates: `["from company_sdk"]`, `["@acme/rpc"]`, `["require('internal-bus')"]`
Bad gates: `["import"]` (too broad), `["client"]` (matches everything)

### detections[].match

Substring searched line by line in files that pass the gate. Include enough context to be unique.

Good matches: `".publish("`, `"= Client("`, `"createClient("`
Bad matches: `"publish"` (too generic), `"("` (matches everything)

### detections[].target_extraction

| Value | Use when | What it extracts |
|-------|----------|-----------------|
| `"none"` | Call doesn't take a target argument | Nothing |
| `"first_string_arg"` | First argument is a string literal | First quoted string on the line |
| `"url_hostname"` | First argument is a full URL | Hostname from the URL |
| `"named_arg:KEY"` | Target is a keyword argument | Value after `KEY=` on the line |

### detections[].confidence

- `"high"` — import gate + match is unambiguous
- `"medium"` — likely correct, could have false positives
- `"low"` — speculative, heuristic-based

### detections[].protocol

Free-form string. Common values: `rest`, `grpc`, `graphql`, `kafka`, `rabbitmq`, `nats`, `mqtt`, `redis`, `postgres`, `mongodb`, `mysql`, `sqs`, `sns`, `modbus`, `opcua`, `kubernetes`
