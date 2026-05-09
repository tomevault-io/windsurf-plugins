---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ThreadHop** is a Textual TUI plus CLI for browsing, searching, and carrying context across Claude Code session transcripts. macOS-only — uses `ps`/`lsof` for active session detection.

The project is expanding from a transcript viewer into a cross-session context manager with SQLite FTS search, project memory, session tagging, and a skill plugin for handoff generation. See `docs/DESIGN-DECISIONS.md` for the full architecture.

## Running

```bash
# TUI
./threadhop
./threadhop --project myproject --days 7

# CLI subcommands (all accept --project / --session)
./threadhop tag <status>            # backlog|in_progress|in_review|done|archived
./threadhop bookmark [kind]         # bookmark|research against latest msg or --message <uuid>
./threadhop todos                   # open TODOs from observations
./threadhop decisions               # decisions extracted by observer
./threadhop observations            # raw observation JSONL, newest first
./threadhop conflicts [--resolved]  # cross-session decision conflicts (reflector)
./threadhop observe [--once|--stop|--stop-all] [--watch-backend auto|poll|fsevents]
```

No build step. The script uses `uv run --script` with PEP 723 inline metadata. Runtime deps: `textual`, `pydantic`. Tests use `pytest`.

## Architecture

Core modules:
- `threadhop` — executable entry point, TUI, argparse routing, CLI handlers, command registry + help overlay
- `db.py` — SQLite schema, migrations, session / bookmark / observation-state / memory helpers, CHECK constraints (ADR-004)
- `models.py` — Pydantic validation boundary for JSONL parsing and DB-row shapes; `Literal` enums mirrored by SQL CHECKs (task #24)
- `indexer.py` — transcript normalization + FTS ingestion; merges assistant streaming chunks by `message.id` (ADR-003)
- `observer.py` — sidecar orchestrator: seek-from-byte-offset, batch-threshold gate, `claude -p --model haiku` extractor, watch-mode with poll/fsevents backends (ADR-018)
- `reflector.py` — reads observation JSONL across sessions in one project, appends `type: "conflict"` entries back into the same file (ADR-020)
- `cli_queries.py` — shared CLI helpers: session-to-project sync, observer catch-up driver for `todos`/`decisions`/`observations`/`conflicts`
- `observation_queries.py` — low-level readers for per-session observation JSONL
- `migration.py` — one-time move of session metadata from `config.json` into SQLite (ADR-001); idempotent, transactional
- `prompts/observer.md`, `prompts/reflector.md` — system prompts spliced with live context before each `claude -p` call

### Key Classes

| Class | Role |
|-------|------|
| `ClaudeSessions(App)` | Main Textual app — layout, refresh loop, keybindings, session discovery, command registry dispatch |
| `TranscriptView(VerticalScroll)` | Parses JSONL, renders conversation as `UserMessage`/`AssistantMessage`/`ToolMessage` widgets; select-mode + bookmark toggle |
| `SessionItem(ListItem)` | Renders one session row: status icon (◐ working / ● active / ○ inactive) + name + age |
| `SearchScreen(ModalScreen)` | FTS5 search across indexed messages |
| `BookmarkBrowserScreen(ModalScreen)` | Pinned-message browser; list → enter jumps to message in transcript (task #18) |

### Data Flow

1. **Discovery**: `_gather_session_data()` runs in a background worker every 5s — scans `~/.claude/projects/**/*.jsonl`, reads first 100 lines for metadata
2. **Active detection**: `_get_active_claude_sessions()` runs `ps -eo pid,args`, finds `claude` processes, resolves CWD via `lsof -a -d cwd -p <pid>`, matches to session IDs
3. **Display**: `_update_session_list()` diffs old/new session lists — full rebuild on change, in-place spinner updates otherwise
4. **Transcript**: `load_transcript()` parses full JSONL via `models.parse_transcript_line`, strips `<system-reminder>` tags, abbreviates tool calls, mounts message widgets
5. **Observer pipeline**: `observer.observe_session()` reads `observation_state.source_byte_offset`, re-uses the cleaned transcript from `indexer.parse_byte_range` (same view the TUI shows), gates on `BATCH_THRESHOLD` new turns, invokes `claude -p --model haiku --permission-mode acceptEdits` with `prompts/observer.md` — the child process appends JSONL into `~/.config/threadhop/observations/<session_id>.jsonl`. Watch-mode loops this with fsevents/poll until `--stop`.
6. **Reflector pipeline**: after enough new observations accumulate, `reflector.reflect_session()` compares the session's decisions against sibling sessions in the same project and appends `type: "conflict"` rows to the same observation JSONL.
7. **Observation CLI**: `todos` / `decisions` / `observations` / `conflicts` run observer catch-up for tracked sessions via `cli_queries`, then read the per-session JSONL. `conflicts --resolved` writes review state into the `conflict_reviews` table instead of mutating append-only JSONL.

### Session State

- **is_active**: A `claude` process is running for this session (matched by session ID in args or CWD)
- **is_working**: Active + recently modified + has pending tool call or last message was from user

### Persistent Config


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [parzival1l/threadhop](https://github.com/parzival1l/threadhop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
