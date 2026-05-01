---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Redd-Archiver is a PostgreSQL-backed archive generator that transforms compressed data dumps from multiple link aggregator platforms (**Reddit**, **Voat**, **Ruqqus**) into browsable static HTML websites with optional server-side full-text search and MCP/AI integration.

**Key Characteristics:**
- **Multi-Platform Support**: Reddit (.zst), Voat (SQL), Ruqqus (.7z)
- Streaming architecture with constant memory usage regardless of dataset size
- PostgreSQL-only backend (DATABASE_URL required)
- Hybrid output: Static HTML for offline browsing + optional Flask search server
- **REST API v1** with MCP/AI optimization (see `docs/API.md`)
- **MCP Server** for Claude Desktop/Claude Code integration (see `mcp_server/README.md`)
- Zero JavaScript design for maximum compatibility

## Deviations from Global Standards

This project intentionally deviates from the global `~/.claude/docs/` conventions in these areas. Do not "fix" these to match global standards without explicit permission.

| Area | Global Standard | This Project | Rationale |
|------|----------------|--------------|-----------|
| Line length | 88 | 120 | HTML string literals, SQL queries, and template paths cause excessive wrapping at 88 |
| Type checker | pyright (standard) | Not yet configured | Legacy codebase; planned addition (see `roadmap/08-pyright-type-checking.md`) |
| Logging | structlog | stdlib `logging` | Predates structlog adoption; migration low priority |
| Project layout | `src/` | Flat (packages at root) | Historical; changing breaks all Docker COPY paths and imports |
| Source control | jj preferred | git only | jj not configured for this repo |
| Pre-commit | Expected | Not yet activated | `pre-commit` in dev deps but no `.pre-commit-config.yaml`; CI gates enforce quality. Planned (see `roadmap/10-pre-commit-hooks.md`) |
| Ruff rules | includes SIM, RUF | Missing SIM, RUF | Being added incrementally (see `roadmap/09-ruff-sim-ruf-rules.md`) |
| Docker Python | Consistent | 3.12 (builder) vs 3.14 (search-server) | Known mismatch; fix planned (see `roadmap/11-docker-python-version-alignment.md`) |

## Build & Run Commands

### Docker Development (Primary Method)

```bash
# Start all services (postgres, builder, search-server, nginx)
sudo docker compose up -d --build

# Run archive generator (Reddit)
sudo docker compose exec reddarchiver-builder python reddarc.py /data \
  --output /output/ \
  --subreddit privacy \
  --comments-file /data/Privacy_comments.zst \
  --submissions-file /data/Privacy_submissions.zst

# Voat (pre-split files)
sudo docker compose exec reddarchiver-builder python reddarc.py /data/voat_split/submissions/ \
  --subverse privacy \
  --comments-file /data/voat_split/comments/privacy_comments.sql.gz \
  --submissions-file /data/voat_split/submissions/privacy_submissions.sql.gz \
  --platform voat \
  --output /output/

# Ruqqus (.7z files)
sudo docker compose exec reddarchiver-builder python reddarc.py /data/ruqqus/ \
  --guild technology \
  --comments-file /data/ruqqus/comments.fx.2021-10-30.txt.sort.2021-11-08.7z \
  --submissions-file /data/ruqqus/submissions.f1.2021-10-30.txt.sort.2021-11-10.7z \
  --platform ruqqus \
  --output /output/

# View logs / health
sudo docker compose logs -f search-server
curl http://localhost/health       # nginx
curl http://localhost:5000/health  # search-server
```

### Deployment Modes

```bash
docker compose up -d                                       # Development (HTTP)
docker compose --profile production up -d                  # HTTPS (Let's Encrypt)
docker compose --profile tor up -d                         # Tor Hidden Service
docker compose --profile production --profile tor up -d    # HTTPS + Tor
```

### Local Development

```bash
export DATABASE_URL="postgresql://user:pass@localhost:5432/reddarchiver"
uv run python reddarc.py /path/to/data --output archive/
uv run python search_server.py
```

### Makefile Shortcuts

```bash
make setup          # uv sync + install pre-commit hooks
make test           # pytest
make test-cov       # pytest with coverage report
make lint           # ruff check
make format         # ruff format
make docker-up      # Start Docker services
make docker-logs    # Tail Docker logs
make clean          # Remove caches and temp files
```

## CLI Quick Reference

| Argument / Flag | Description |
|----------------|-------------|
| `input_dir` | (Required) Directory containing data files |
| `--output/-o DIR` | Output directory (default: `redd-archive-output`) |
| `--import-only` | Stream to PostgreSQL only (no HTML) |
| `--export-from-database` | Generate HTML from existing DB only (no import) |
| `--subreddit/-s NAME` | Reddit subreddit(s), comma-separated |
| `--subverse NAME` | Voat subverse(s), comma-separated |
| `--guild NAME` | Ruqqus guild(s), comma-separated |
| `--platform TYPE` | Force platform: `auto\|reddit\|voat\|ruqqus` |
| `--comments-file PATH` | Path to comments file (.zst/.sql.gz/.7z) |
| `--submissions-file PATH` | Path to submissions file (.zst/.sql.gz/.7z) |
| `--min-score N` | Minimum post score threshold |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [19-84/redd-archiver](https://github.com/19-84/redd-archiver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
