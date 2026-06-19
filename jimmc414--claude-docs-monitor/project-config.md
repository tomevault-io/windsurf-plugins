---
trigger: always_on
description: Single-file Python tool that monitors Claude Code documentation at `code.claude.com/docs/` for changes. Fetches all pages, stores snapshots in SQLite, computes unified diffs, generates HTML/Markdown reports, produces AI-powered change digests via `claude -p`, and maintains a persistent change intelligence database with structured AI classifications queryable by category, severity, page, keyword, and date range.
---

# Claude Docs Monitor — AI Agent Reference

## Purpose

Single-file Python tool that monitors Claude Code documentation at `code.claude.com/docs/` for changes. Fetches all pages, stores snapshots in SQLite, computes unified diffs, generates HTML/Markdown reports, produces AI-powered change digests via `claude -p`, and maintains a persistent change intelligence database with structured AI classifications queryable by category, severity, page, keyword, and date range.

## Architecture

### Single-File Design

Everything lives in `claude_docs_monitor.py` (~2150 lines). No package structure, no setup.py. One required dependency (`httpx[http2]`), two optional (`rich`, `mcp`). An optional `mcp_server.py` (~220 lines) provides read-only MCP access to the database.

### Layers (top to bottom)

```
CLI (build_parser, main)
  → Commands (cmd_check, cmd_digest, cmd_history, cmd_diff, cmd_urls, cmd_dump, cmd_rebuild_history, cmd_query, cmd_backfill)
    → Change Intelligence (_run_structured_classification, _create_gh_issues)
    → Report Generation (generate_md_report, generate_html_report, append_*_history)
    → Display (print_summary, print_diffs — Rich or plain text)
    → HTTP (fetch_url, fetch_all — async httpx with semaphore + retry)
    → Database (init_db, store_*, get_*, query_change_events — SQLite append-only)
    → Utilities (sha256, normalize, compute_diff, parse_index, is_html_diff, _parse_relative_date)

MCP Server (mcp_server.py — optional, read-only)
  → Tools (query_changes, get_page_snapshots, get_diff, search_pages)
  → Resources (docs://pages/{name}, docs://digest, docs://report)
  → Imports from claude_docs_monitor (init_db, query_change_events, get_page_history, etc.)
```

### Database Schema

Three tables in `data-claude/snapshots.db`:

**`index_snapshots`** — tracks the llms.txt index itself (append-only):
- `id` INTEGER PK, `fetched_at` TEXT, `content` TEXT, `hash` TEXT, `urls_json` TEXT

**`page_snapshots`** — one row per fetch per URL (append-only):
- `id` INTEGER PK, `url` TEXT, `fetched_at` TEXT, `content` TEXT, `hash` TEXT, `status_code` INTEGER, `duration_ms` REAL, `error` TEXT
- Indexes: `idx_page_url` (url), `idx_page_fetched` (fetched_at)

**`change_events`** — AI-classified change metadata (append-only):
- `id` INTEGER PK, `run_timestamp` TEXT, `url` TEXT, `page_name` TEXT, `event_type` TEXT, `category` TEXT, `severity` TEXT, `summary` TEXT, `details` TEXT, `action_required` TEXT, `tags_json` TEXT, `diff_text` TEXT, `gh_issue_url` TEXT, `created_at` TEXT
- Indexes: `idx_ce_run` (run_timestamp), `idx_ce_url` (url), `idx_ce_category` (category), `idx_ce_severity` (severity)
- Categories: feature, breaking, deprecation, clarification, flag_change, bugfix
- Severity levels: high, medium, low

### Data Flow: `check` command

1. Fetch `INDEX_URL` (llms.txt) → parse URLs → store index snapshot
2. Compare URL set against previous index → detect added/removed pages
3. `fetch_all()`: async HTTP/2, 5 concurrent connections, 3 retries with exponential backoff
4. For each page: SHA-256 hash comparison against last snapshot → compute unified diff if changed
5. Filter HTML noise diffs (>50% HTML tags) unless `--include-html`
6. Store all snapshots → generate reports (md + html + history) → dump pages to disk

### Data Flow: `digest` command

1. Read `data-claude/report.md` → check for changes (| Changed | 0 |)
2. Extract `## Diffs` section to minimize tokens
3. **Phase 1 — Structured classification**: Pipe diffs to `claude -p` with `--output-format json --json-schema` → parse JSON → store events in `change_events` table (skips if already classified for this run)
4. **Phase 2 — Text digest**: Pipe diffs to `claude -p --output-format text` → write `digest.md` and `digest.html`
5. **Phase 3 — GitHub issues** (optional, `--gh-issue`): Create issues via `gh` CLI for breaking changes
6. Strip `CLAUDECODE` env var to allow nested invocation

### Key Design Decisions

- **Hash-before-diff**: SHA-256 comparison is O(1); only compute expensive diffs when content changed
- **Status code tracking**: 200→non-200 treated as a change even without content diff
- **HTML noise filter**: `is_html_diff()` counts HTML/JS patterns in changed lines; >50% threshold suppresses the diff
- **First-run detection**: If first URL has no prior snapshot, treat entire run as baseline (no "added" reports)
- **Index tracking**: Stores `urls_json` per run to detect when Anthropic adds/removes doc pages
- **Digest via CLI**: Uses `subprocess.run(["claude", "-p", ...])` not the SDK — works with Max OAuth automatically
- **History reconstruction**: `rebuild-history` walks index snapshots chronologically, uses time windows to group page fetches per run
- **Structured classification via JSON schema**: Uses `claude -p --output-format json --json-schema` for deterministic structured output; graceful fallback if classification fails (text digest still runs)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jimmc414/claude_docs_monitor](https://github.com/jimmc414/claude_docs_monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
