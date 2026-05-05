---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Status

Aguara v0.14.2 (2026-04-18). 189 rules, 13 categories, 4 analysis layers, ~630 tests, 0 lint issues.

Distribution: install.sh (mandatory checksum verification, bounded curl + retry), Homebrew tap, Docker (GHCR, signed at digest with Cosign + SBOM + SLSA provenance attestations), GoReleaser (releases signed via Cosign keyless, SPDX SBOM per archive, `-trimpath` for reproducibility), GitHub Action, go install.

GitHub: 48 stars, 6 forks. 0 open PRs, 0 open issues. 7 awesome-list PRs pending review in external repos. 7 forks on garagon account (pending cleanup after awesome-list PRs resolve).

Pending improvements: pattern matcher performance (~770ms/file), WASM build (cmd/wasm/ incomplete), adoption/marketing.

Internal docs (gitignored) in `DOCS/` as an Obsidian vault. See `DOCS/Aguara/CONVENTIONS.md` for vault rules and `DOCS/Aguara/00 Dashboard.md` for the entry point.

Key locations:
- Dashboard (MOC): `DOCS/Aguara/00 Dashboard.md`
- Conventions: `DOCS/Aguara/CONVENTIONS.md` - frontmatter schema, types, statuses, tags, naming
- Product versions: `DOCS/Aguara/product/v0.10.0/_index.md` (version hub with links to all generated content; v0.11.0–v0.14.0 hubs not yet created)
- Distribution channels: `DOCS/Aguara/distribution/<channel>/_index.md` (each has submission log)
- Growth plan: `DOCS/Aguara/product/roadmap.md`
- Templates: `DOCS/Aguara/templates/` (channel.md, product-version.md)
- Demo video project: `/Users/dev/Dev/videos/aguara-demo/` (Remotion, React-based)
- Oktsec docs (separate project): `DOCS/Oktsec/`

## Build & Test Commands

```bash
make build          # Production binary with version/commit ldflags
make test           # go test -race -count=1 ./...
make lint           # golangci-lint run ./...
make vet            # go vet ./...
make fmt            # gofmt -w .
make run ARGS="scan ./path"  # Development run

# Single package test
go test -race -count=1 ./internal/rules/...

# Single test function
go test -race -count=1 -run TestFunctionName ./internal/engine/pattern/...

# Benchmarks
go test -bench=. -benchtime=3x ./internal/engine/pattern/
go test -bench=. -benchtime=3x ./internal/engine/nlp/
go test -bench=. -benchtime=3x ./internal/scanner/
```

CI requires: `make build && make test && make vet && make lint` all passing.

## Architecture

Aguara is a deterministic static security scanner for AI agent skills and MCP servers. No LLM, no network calls. Go 1.25, module `github.com/garagon/aguara`.

### Public Library API (`aguara.go`, `options.go`)

Root package re-exports types from `internal/types` and exposes: `Scan()`, `ScanContent()`, `ListRules()`, `ExplainRule()`, `Discover()`. Used by external consumers like `aguara-mcp`. Functional options pattern (`WithMinSeverity()`, `WithWorkers()`, etc.).

### Analysis Pipeline (4 layers, run sequentially per file)

1. **Pattern Matcher** (`internal/engine/pattern/`) - Regex/contains matching against compiled rules. 8 decoders (base64, hex, URL encoding, Unicode escapes, HTML entities, hex escapes, base32, C-style octal escapes) for encoded evasion detection. Markdown code-block severity downgrade. Dynamic confidence based on pattern hit ratio.
2. **NLP Analyzer** (`internal/engine/nlp/`) - Goldmark AST walker for markdown; JSON/YAML string extractor for structured files. Keyword classification with proximity weighting. Detects prompt injection, authority claims, credential+exfil combos.
3. **ToxicFlow** (`internal/engine/toxicflow/`) - Single-file taint tracking + cross-file correlation (`crossfile.go`). Detects dangerous capability combinations within and across files in the same directory. Flat-dir filter (>50 files) prevents FPs on registries.
4. **Rug-Pull** (`internal/engine/rugpull/`) - SHA256-based tool description change detection. CLI via `--monitor`, library via `WithStateDir()`.

All four implement the `Analyzer` interface (`internal/scanner/analyzer.go`): `Name() string` + `Analyze(ctx, *Target) ([]Finding, error)`.

### Key Package Relationships

- `internal/types/` - Lowest layer. `Finding`, `Severity`, `ScanResult`. No internal imports. **`Severity` is `type int`, serializes as a number in JSON (0=INFO, 1=LOW, 2=MEDIUM, 3=HIGH, 4=CRITICAL), NOT a string.**
- `internal/rules/` - YAML to CompiledRule. `builtin/` embeds 12 YAML files via `go:embed`.
- `internal/scanner/` - Orchestrator. Discovers files, spawns workers, runs analyzers, applies inline ignore filtering, aggregates results. Imports `meta/` for post-processing.
- `internal/meta/` - Dedup, scoring, correlation. Imports `types/` only (NOT `scanner/` - this prevents import cycles).
- `internal/output/` - Formatters (terminal, JSON, SARIF, markdown) implementing `Formatter` interface. Markdown header is `## Aguara Security Scan`, not `# Aguara Scan Report`.
- `internal/config/` - `.aguara.yml` loader. Supports `disable_rules` list and `rule_overrides` map.
- `discover/` - MCP client auto-discovery. `Scan()`, `FormatTree()`, `FormatMarkdown()`. Public package.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [garagon/aguara](https://github.com/garagon/aguara) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
