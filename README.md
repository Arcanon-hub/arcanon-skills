# Arcanon Skills

AI assistant skills for the [Arcanon Scanner](https://github.com/Arcanon-hub/arcanon-scanner) — a Rust CLI that statically analyzes codebases to detect services, endpoints, and connections.

## Install

```bash
npx skills add arcanon-hub/arcanon-skills
```

Install a specific skill:

```bash
npx skills add arcanon-hub/arcanon-skills --skill arcanon-scan
```

Target a specific agent:

```bash
npx skills add arcanon-hub/arcanon-skills --agent claude-code cursor
```

## Skills

| Skill | Description |
|-------|-------------|
| **arcanon-scan** | Run the scanner, analyze results, find missed connections, write `.arcanon.toml` patterns |
| **arcanon-config** | Configure `.arcanon.toml` — patterns, exclusions, plugin settings |
| **arcanon-improve** | Recommend scanner source code or CDN pattern changes for gaps that patterns can't solve |

## Prerequisites

Install the Arcanon Scanner:

```bash
curl -fsSL https://arcanon.dev/install.sh | sh
```

## License

MIT
