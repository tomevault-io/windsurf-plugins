---
trigger: always_on
description: > **SBI** (Secure Base Images) is a CLI tool that scans container base images and, in the default nightly workflow/config, performs vulnerability scanning of images from MCR (Microsoft Container Registry), then generates ranked recommendation reports.
---

# AGENTS.md — LLM Context for SBI

> **SBI** (Secure Base Images) is a CLI tool that scans container base images and, in the default nightly workflow/config, performs vulnerability scanning of images from MCR (Microsoft Container Registry), then generates ranked recommendation reports.

## Project Overview

- **Module:** `github.com/microsoft/sbi`
- **Language:** Go 1.26, built with `task build`
- **CLI framework:** Cobra (`github.com/spf13/cobra`)
- **Database:** SQLite via `modernc.org/sqlite` (pure Go, no CGO)
- **Binary name:** `sbi`

## Repository Structure

```text
*.go (root)                   # Cobra CLI commands (scan, report, reset-db)
pkg/
  domain/                     # Data models (ImageRecord, Language, Vulnerability, etc.)
  infrastructure/
    database/                 # SQLite schema, persistence, ranked queries
      schema.go               # Table definitions (images, languages, vulnerabilities, etc.)
      repository.go           # InsertImage (upsert), QueryTopImages, QueryLanguages, QueryAllImageDetails
    scanner/
      analyzer.go             # Orchestrates: pull -> inspect -> syft -> trivy -> verify
      syft.go                 # SBOM parsing, language detection from packages
      trivy.go                # Vulnerability scanning
      registry.go             # Tag discovery from container registries
      docker.go               # Docker pull, inspect, run commands in containers
    report/
      markdown.go             # Markdown report generation
      json.go                 # JSON summary report generation
      json_detail.go          # Detailed per-image JSON report (packages, CVEs, languages)
  usecase/
    pipeline.go               # Top-level orchestration: config -> discover -> scan -> report
config/
  repositories.json           # Image sources for nightly scans
  smoke-test/repositories.json # Minimal config for CI smoke tests
docs/
  daily_recommendations.md    # Generated nightly report (committed to repo)
  daily_recommendations.json  # Generated JSON report
```

## Build, Test, Lint

All commands use [Task](https://taskfile.dev/) (see `Taskfile.yml`).
**Always run `task lint` before creating a PR** — CI runs linting on all PRs and will fail on errors.

```bash
task build          # Build binary to bin/{OS}-{ARCH}/sbi
task test           # Unit tests with coverage
task test:short     # Fast tests without coverage
task lint           # All linters: golangci-lint, markdown, YAML, govulncheck
task lint:go        # golangci-lint with auto-fix
task lint:md        # Markdown linting (uses markdownlint-cli)
task lint:yaml      # YAML validation
task all            # deps -> lint -> test -> build
task scan           # Build + run full nightly scan
task report         # Generate reports from existing database
task clean          # Remove build/coverage artifacts
```

## CLI Usage

```bash
# Install
go install github.com/microsoft/sbi@latest

# Scan images and generate report
sbi scan \
  --database azure_linux_images.db \
  --config-dir config \
  --output docs/daily_recommendations.md \
  --max-tags 10 \
  --top-n 10 \
  --comprehensive \
  --verbose

# Generate report from existing DB
sbi report --database azure_linux_images.db

# Reset database
sbi reset-db --database azure_linux_images.db
```

**Key flags:**

- `--top-n N` — number of top images per language per base OS in markdown report (default 10, 0 = all)
- `--json-top-n N` — number of top images per language per base OS in JSON report (default 20, 0 = all)
- `--max-tags N` — limit tags per repository (0 = all)
- `--comprehensive` — enable secrets + misconfiguration scanning
- `--detailed` — generate a detailed per-image JSON report with full package/vulnerability/language data
- `--update-existing` — rescan images already in the database
- `--no-cleanup` — keep Docker images after scanning

## Scan Pipeline Flow

```text
config/repositories.json
  |
  |-- Repository entries (no ':') -> GetTags() from registry API -> FilterTags() -> LimitTags()
  |-- Single image entries (has ':') -> scan directly
  |
  v
For each image:
  1. docker pull
  2. docker inspect          -> size, layers, digest, created date
  3. syft <image> -o json    -> SBOM: languages, packages, capabilities
  4. trivy image <image>     -> vulnerabilities (+ secrets/misconfig if --comprehensive)
  5. mergeLanguages()        -> combine Syft results + image-name detection
  6. verifyRuntimes()        -> run version commands inside container
  7. applyRuntimeVersions()  -> update with verified versions
  8. InsertImage() (upsert)  -> store in SQLite
  |
  v
Generate reports:
  - docs/daily_recommendations.md
  - docs/daily_recommendations.json
```

## Image Configuration Format

`config/repositories.json` supports two entry types:

```json
{
  "defaults": { "registry": "mcr.microsoft.com", "maxTags": 0 },
  "tagFilter": {
    "skipExact": ["latest", "dev", "nightly"],
    "excludeKeywords": ["debug", "test", "experimental", "arm", "amd", "azl"],
    "excludePatterns": ["(?i)[-.]?(alpha|beta|rc|preview)...", "^\\d+\\.\\d+\\.\\d{8}$"],

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/sbi](https://github.com/microsoft/sbi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
