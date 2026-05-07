---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MemBrowse GitHub Actions is a collection of GitHub Actions for analyzing memory usage in embedded firmware and uploading reports to the MemBrowse SaaS platform. The repository contains two main actions:

- **pr-action**: Analyzes memory usage in pull requests and push events
- **onboard-action**: Performs historical analysis across multiple commits for onboarding

## Command-Line Interface

### Unified CLI (`membrowse`)

MemBrowse provides a single `membrowse` command with subcommands for different operations:

#### `membrowse report` - Generate Memory Footprint Reports

Core analysis tool that generates memory footprint reports from ELF files and linker scripts.

**Local Mode (Default)** - Generate memory report without Git metadata or uploading:
```bash
# Minimal usage - outputs human-readable report (only warnings/errors shown)
membrowse report firmware.elf "linker.ld"

# Output as JSON instead
membrowse report firmware.elf "linker.ld" --json

# Save JSON to file
membrowse report firmware.elf "linker.ld" --json > report.json

# With verbose output to see progress
membrowse -v INFO report firmware.elf "linker.ld"

# Multiple linker scripts
membrowse report firmware.elf "mem.ld sections.ld"
```

**Note**: By default, output is human-readable and INFO-level messages are logged (one line per upload). Use `-v DEBUG` before the subcommand for detailed progress, or `-v WARNING` to suppress INFO messages.

**Upload Mode** - Upload report to MemBrowse platform:
```bash
# Upload without Git metadata
membrowse report firmware.elf "linker.ld" --upload \
    --api-key "$API_KEY" \
    --target-name "esp32" \
    --api-url "https://membrowse.com"

# Upload with Git metadata (manual)
membrowse report firmware.elf "linker.ld" --upload \
    --api-key "$API_KEY" \
    --target-name "stm32f4" \
    --api-url "https://membrowse.com" \
    --commit-sha "abc123" \
    --branch-name "main" \
    --repo-name "my-repo"
```

**GitHub Actions Mode** - Automatically detect Git metadata from GitHub environment:
```bash
# Auto-detects Git metadata from GitHub environment
membrowse report firmware.elf "linker.ld" --upload --github \
    --target-name "esp32" \
    --api-key "$API_KEY" \
    --api-url "https://membrowse.com"
```

**Modes:**
- **Local mode (default)**: Generates human-readable report to stdout (use `--json` for JSON output)
- **Upload mode**: Uploads report to MemBrowse platform (requires `--upload` flag), auto-detects Git metadata from local git
- **GitHub mode**: Use `--github` with `--upload` to detect Git metadata from GitHub Actions environment variables instead of local git

**Key Features:**
- Git metadata is auto-detected by default when uploading (use `--no-git` to disable)
- Target name only required when uploading
- All Git metadata is optional

#### `membrowse summary` - Fetch Memory Summary for a Commit

Fetches a consolidated memory summary for a given commit from the MemBrowse API and renders it as markdown (for PR comments).

```bash
# Render summary as markdown (default template)
membrowse summary <commit-sha> --api-key "$API_KEY"

# Output raw JSON response
membrowse summary <commit-sha> --api-key "$API_KEY" --json

# Use custom Jinja2 template
membrowse summary <commit-sha> --api-key "$API_KEY" --template path/to/template.j2

# Custom API URL
membrowse summary <commit-sha> --api-key "$API_KEY" --api-url "https://api.membrowse.com"
```

Used by the `comment-action` to post consolidated PR comments that summarize memory changes across all targets for a commit.

#### `membrowse onboard` - Historical Analysis

Analyzes memory footprints across multiple historical commits and uploads them to the MemBrowse platform:
```bash
# Analyze last 50 commits with linker scripts
membrowse onboard 50 "make clean && make" build/firmware.elf \
    stm32f4 "$API_KEY" https://membrowse.com --ld-scripts "linker.ld"

# Without linker scripts (uses default Code/Data regions)
membrowse onboard 50 "make clean && make" build/firmware.elf \
    stm32f4 "$API_KEY"

# ESP-IDF project (API URL is optional, defaults to https://api.membrowse.com)
membrowse onboard 25 "idf.py build" build/firmware.elf \
    esp32 "$API_KEY" --ld-scripts "build/esp-idf/esp32/esp32.project.ld"

# Binary search mode: minimize builds by detecting memory change points
# Only builds endpoints and midpoints, marks unchanged ranges as identical
membrowse onboard 50 "make clean && make" build/firmware.elf \
    stm32f4 "$API_KEY" --binary-search

# Dry-run mode: build and analyze but skip uploading (for testing)
membrowse onboard 50 "make clean && make" build/firmware.elf \
    stm32f4 "$API_KEY" --dry-run

# Combine dry-run with binary search
membrowse onboard 50 "make clean && make" build/firmware.elf \
    stm32f4 "$API_KEY" --binary-search --dry-run
```

### Performance Options

#### --skip-line-program flag

Skips DWARF line program processing for faster analysis at the cost of reduced source file coverage.

**When to use:**
- ✅ Build speed is critical (CI/CD, iterative development)
- ✅ 88% coverage acceptable (ARM) or 65% (ESP32)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [membrowse/membrowse-action](https://github.com/membrowse/membrowse-action) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
