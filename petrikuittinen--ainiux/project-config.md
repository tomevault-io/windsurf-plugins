---
trigger: always_on
description: Repository-level instructions for AI coding agents. Treat this as current project guidance. The user's latest explicit instruction always wins. For milestones use `PLANS.md`; for release history use `docs/version-history.md`; for user-facing usage start with `README.md` and `docs/README.md`; for design rationale use `docs/decisions.md`; for open work use `TODO.md`.
---

# AGENTS.md

Project: `ainiux`

Repository-level instructions for AI coding agents. Treat this as current project guidance. The user's latest explicit instruction always wins. For milestones use `PLANS.md`; for release history use `docs/version-history.md`; for user-facing usage start with `README.md` and `docs/README.md`; for design rationale use `docs/decisions.md`; for open work use `TODO.md`.

## Mission

Build and maintain `ainiux`: a fast, portable command-line and terminal chat client for OpenAI and OpenAI-compatible APIs, with a first-class standalone editor, document conversion, benchmarks, and judge grading.

The program must stay excellent as a scriptable CLI. Keep the core engine independent from UI surfaces so the same request, provider, streaming, persistence, runtime/job, cancellation, memory-management, and error-handling code is reused by:

- non-interactive CLI chat
- document extraction / conversion
- REPL
- full-screen chat TUI
- standalone editor (with optional AI assist)
- benchmark runner
- grade (judge) pass
- future local OpenAI-compatible server mode
- postponed browser web UI
- local agent mode with session-scoped Act/Plan task modes

## Current product snapshot

Status: **v1.16 plus an unreleased native Windows parity target** (see `README.md`, `docs/windows.md`, `docs/agent.md`, and `PLANS.md`). One-shot (`run` / `--run` / `-r`) and interactive (`agent` / `--agent` / `-a`) local agent modes are landed with workspace writes, multi-turn project sessions (`.ainiux-pr/`), compact live tool activity, provider-supplied reasoning previews in interactive agent history, three-strategy transcript-preserving compaction, retained row-diff terminal rendering, punctuation-aware Markdown highlighting, chat↔editor↔agent cycling, project-persisted Confirm/Smart/Yolo permissions, OpenRouter/OpenAI/DeepSeek credit display, interactive Guard approvals, and session-scoped Act/Plan task modes. Live tool rows update in place, while display-only `notice` and `thinking` rows remain outside provider context. One-shot planning is available through `plan`, `--plan`, and `--plan-file`; Plan retains research tools but code-enforces planning-document-only writes. User profile stays `~/.ainiux/` (chat DB/media). The **v1.1** code index is a lightweight definitions-only index across all scanner languages, with static declaration importance and mutation-aware persistence. The Windows target remains unreleased until its native parity gate passes. Local server mode is deferred behind v1.1, image generation moves to v1.2, and browser web UI remains postponed.

### Implemented modes

| Mode | Entry | Notes |
| --- | --- | --- |
| One-shot CLI chat | default + `-p` / `--prompt-file` | stdout = model output; stderr = status/errors |
| Document extract | `--input` / `--fetch-url` without chat prompt | text/Markdown/HTML conversion; image input for chat path |
| List models | `--list-models` | provider `/models` |
| REPL | `--repl` / `-i` | line-oriented interactive |
| Chat TUI | `--chat` (`--tui` alias) | non-blocking alternate-screen UI; SQLite threads |
| Standalone editor | `--editor [path]` | multi-buffer piece-table editor; optional AI assist |
| Benchmark | `benchmark` / `--benchmark` | concurrent JSONL dataset runner |
| Grade | `--grade` | second-pass judge scoring of benchmark results (not combined with `--benchmark`) |
| Interactive agent | `agent` / `--agent` / `-a` | **Separate mode from `--chat`**. Project-local `.ainiux-pr/`; Act by default; `/plan` and `/act` switch session task policy; `/goal` sets a persistent completion condition (`goal_met`); shared TUI shell + selectors |
| One-shot agent | `run` / `--run` / `-r` / `--run-file` | headless Act mode; compact tool lines on stderr; stdout = final answer |
| One-shot plan | `plan` / `--plan` / `--plan-file` | headless Plan mode; read/research tools plus planning-document-only writes |
| Security review | `--security-review` | headless read-only whole-project review |
| Code index | `--index-code` / `--print-index` / `--clear-index` | project-local `.ainiux-pr/index.sqlite` |

### Implemented capabilities agents must respect

- Built-in provider registry and aliases; Chat Completions; text-only OpenAI Responses API (`--api responses`, `--responses`, `openai_responses`)
- Catalog-selected provider reasoning mapping (`--reasoning auto|VALUE|TOKENS`)
- `--provider none` offline profile for conversion/editor without a model endpoint
- Credential lookup from env / key file / stdin; redaction in logs and artifacts
- JSON chat import/export (`--save-chat` / `--load-chat`); SQLite-backed TUI chat library at `~/.ainiux/ainiux.db`
- Cancellable runtime jobs; libcurl HTTP + incremental SSE streaming
- Request-only context policies; full transcript preserved on disk
- Bounded text/HTML/Markdown attachments; JPEG/PNG/GIF image input (Chat Completions)
- Safe URL fetching; web search (`--search`, `/search`) with API providers and keyless fallbacks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [petrikuittinen/ainiux](https://github.com/petrikuittinen/ainiux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
