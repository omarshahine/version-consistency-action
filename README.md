# Version Consistency Action

A GitHub composite action that verifies version strings across multiple JSON manifest files agree within each declared agreement set. Useful for plugins/packages where the same version must appear in `package.json`, `plugin.json`, a marketplace manifest, an OpenClaw manifest, etc.

## Why

Large plugin repos often carry the same version in 3–6 different files. Any one of them can silently drift when someone bumps "the" version but forgets the others. A release tag ends up shipping the *old* version in the file consumers actually read. This action enforces agreement at PR time so drift never lands on `main`.

## Usage

```yaml
name: Version Consistency

on:
  push:
    branches: [main]
  pull_request:

jobs:
  version-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: omarshahine/version-consistency-action@v1
        with:
          sets: |
            - name: "Claude Code plugin"
              sources:
                - .claude-plugin/plugin.json:.version
                - .claude-plugin/marketplace.json:.plugins[0].version
            - name: "OpenClaw plugin"
              sources:
                - openclaw/package.json:.version
                - openclaw/openclaw.plugin.json:.version
```

Each set is independent. Sources within a set must all resolve to the same version; different sets can hold different versions (e.g., a Claude Code plugin at `4.12.0` alongside an OpenClaw plugin at `2.0.0` in the same repo).

## Input

### `sets` (required)

A YAML list of agreement sets. Each set is a mapping with:

- **`name`** (string) — label for the set, used in output.
- **`sources`** (list of strings) — `file:jq-path` pairs. File paths are relative to the repo root. If the jq path is omitted, it defaults to `.version`.

## Output

Prints a table per set and exits:
- **0** if every set's sources agree
- **1** if any set has drift, a missing file, or a jq lookup failure
- **2** on malformed input or missing `jq`/`python3`

Example passing run:

```
=== Claude Code plugin ===
  .claude-plugin/plugin.json              3.7.1
  .claude-plugin/marketplace.json         3.7.1
  OK: all agree on 3.7.1

=== OpenClaw plugin ===
  openclaw/package.json                   3.7.1
  openclaw/openclaw.plugin.json           3.7.1
  OK: all agree on 3.7.1
```

Example failing run:

```
=== Claude Code plugin ===
  .claude-plugin/plugin.json              3.7.1
  .claude-plugin/marketplace.json         3.6.0
  FAIL: Claude Code plugin versions disagree: ['3.6.0', '3.7.1']
```

## Requirements

Runs on any Linux GitHub runner. Depends on `jq`, `python3`, and `PyYAML` — all preinstalled on `ubuntu-latest` as of early 2026.

## Versioning

Use `@v1` to track the major version (automatic minor/patch updates). Pin to a specific tag like `@v1.0.0` if you want fully reproducible CI.

## License

MIT
