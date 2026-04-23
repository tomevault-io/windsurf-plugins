---
trigger: always_on
description: `kb` is a single-binary Go CLI for building and maintaining topic-based knowledge bases in the Karpathy KB pattern. It handles the non-LLM workflow: topic scaffolding, multi-source ingestion, structural linting, codebase analysis, QMD indexing/search, and KB-oriented inspection commands.
---

# AGENTS.md

## Project Overview

`kb` is a single-binary Go CLI for building and maintaining topic-based knowledge bases in the Karpathy KB pattern. It handles the non-LLM workflow: topic scaffolding, multi-source ingestion, structural linting, codebase analysis, QMD indexing/search, and KB-oriented inspection commands.

**Reference TypeScript source:** `~/dev/projects/kodebase`

## Source of Truth

- KB pivot tech spec: `.compozy/tasks/kb-pivot/_techspec.md`
- KB pivot task tracker: `.compozy/tasks/kb-pivot/_tasks.md`
- KB pivot workflow memory: `.compozy/tasks/kb-pivot/memory/`

## Critical Rules

- `make verify` is the non-negotiable completion gate: `fmt -> lint -> test -> build -> boundaries`.
- `make lint` must report zero findings.
- Use `go get` for dependency changes.
- Never use destructive git restore/reset/checkout/clean/rm commands without explicit approval.
- Prefer local repository inspection over web search for codebase questions.

## Build Commands

```bash
make verify              # fmt -> lint -> test -> build -> boundaries
make fmt                 # gofmt over repository Go files
make lint                # golangci-lint v2
make test                # unit tests with -race via gotestsum
make test-integration    # unit + integration tests with -race and -tags integration
make build               # build ./... and bin/kb with ldflags
make deps                # go mod tidy
make help                # mage target list
```

## Package Layout

| Path | Responsibility |
| --- | --- |
| `cmd/kb` | Program entrypoint for the `kb` binary |
| `internal/cli` | Cobra root, subcommands, flag resolution, and command I/O |
| `internal/topic` | Topic scaffolding, listing, and topic metadata lookup |
| `internal/ingest` | Ingest orchestration, frontmatter assembly, raw writes, and log entries |
| `internal/convert` | Converter registry and format-specific file converters |
| `internal/firecrawl` | Firecrawl REST client for `kb ingest url` |
| `internal/youtube` | YouTube transcript extraction and OpenRouter STT fallback |
| `internal/frontmatter` | Shared frontmatter parsing and generation helpers |
| `internal/lint` | KB structural lint engine and report rendering |
| `internal/generate` | Codebase-to-KB pipeline used by `kb ingest codebase` and the hidden legacy `generate` alias |
| `internal/scanner` | Source discovery and ignore handling for codebase ingest |
| `internal/adapter` | Tree-sitter parsing adapters |
| `internal/graph` | Graph normalization for codebase snapshots |
| `internal/metrics` | File, symbol, directory, and smell metrics |
| `internal/vault` | Topic path helpers, render/write logic, vault reads, and inspect snapshot loading |
| `internal/qmd` | QMD subprocess integration for index and search |
| `internal/output` | Table, JSON, and TSV formatting |
| `internal/models` | Shared domain models and interfaces |
| `internal/config` | TOML config plus env-backed runtime overrides |
| `internal/logger` | Slog logger setup |
| `internal/version` | Build metadata surfaced by `kb version` |

## CLI Commands

| Command | Purpose |
| --- | --- |
| `kb topic new <slug> <title> <domain>` | Scaffold a new knowledge base topic |
| `kb topic list` | List scaffolded topics |
| `kb topic info <slug>` | Show metadata for one topic |
| `kb ingest url <url> --topic <slug>` | Scrape a web URL and ingest into a topic |
| `kb ingest file <path> --topic <slug>` | Convert a local file and ingest into a topic |
| `kb ingest youtube <url> --topic <slug>` | Extract a YouTube transcript and ingest into a topic |
| `kb ingest codebase <path> --topic <slug>` | Analyze a codebase and ingest artifacts into a topic |
| `kb ingest bookmarks <path> --topic <slug>` | Ingest a bookmark-cluster markdown file into a topic |
| `kb lint [<slug>]` | Check one topic for structural KB issues |
| `kb inspect smells` | List smell signals for symbols and files |
| `kb inspect dead-code` | List dead exports and orphan files |
| `kb inspect complexity` | Rank functions by cyclomatic complexity |
| `kb inspect blast-radius` | Rank symbols by blast radius |
| `kb inspect coupling` | Rank files by instability |
| `kb inspect symbol <name>` | Find symbols by case-insensitive substring |
| `kb inspect file <path>` | Resolve one source file by exact path |
| `kb inspect backlinks <name-or-path>` | Show inbound relations for a file or symbol |
| `kb inspect deps <name-or-path>` | Show outgoing relations for a file or symbol |
| `kb inspect circular-deps` | List detected circular dependency cycles |
| `kb search <query>` | Query a vault through QMD hybrid, lexical, or vector modes |
| `kb index` | Create or update a QMD collection for a topic |
| `kb version` | Print build version metadata |

### Command Notes

- `topic` subcommands share the root `--vault` flag for vault path resolution.
- `ingest` subcommands require `--topic <slug>` to identify the target topic.
- `ingest codebase` accepts `--include`, `--exclude`, `--semantic`, `--progress`, and `--log-format`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [compozy/kb](https://github.com/compozy/kb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
