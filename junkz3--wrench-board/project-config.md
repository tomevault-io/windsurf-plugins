---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

`wrench-board` is an agent-native diagnostics workbench for board-level
electronics repair. Claude drives a multi-panel UI (boardview, knowledge
graph, memory bank, diagnostic chat) through tool calls, in response to a
microsoldering technician's natural-language questions. Two LLM paths run the
product: a stateless **knowledge factory** builds per-device packs offline,
and a stateful **diagnostic conversation** runs the live repair session.

## Hard rules — NEVER violate

1. **All code written from scratch.** Never copy from any external codebase.
2. **Apache 2.0** is the license for all code in this repo.
3. **Permissive dependencies only** (MIT, Apache 2.0, BSD). Never pull in
   GPL, AGPL, or LGPL packages.
4. **Open hardware only *in the repo*.** No proprietary schematics or
   boardviews committed to this repository — no Apple, Samsung, ZXW or
   WUXINJI content in `board_assets/`, `web/boards/`, test fixtures, or
   any other tracked path. **Runtime is a separate matter.** The product
   is a pro microsoldering workbench, so the technician must be able to
   run a diagnostic on *any* device at runtime. Knowledge packs built by
   the pipeline (LLM-driven research under `memory/{slug}/*.json`) are
   unrestricted by device brand — iPhone, Galaxy, ThinkPad, Framework,
   whatever hits the bench. The technician may also upload their own
   schematic PDF or boardview and those land under `memory/{slug}/` so
   subsequent repairs on the same device reuse the pack + schematic +
   boardview automatically. What stays out of this repo is proprietary
   *source* material — not the technician's runtime workflow. The entire
   `memory/` tree (except `.gitkeep`) is gitignored for this reason.
5. **No hallucinated component IDs.** Defense in depth, two layers.
   (1) Tool discipline: every refdes the agent surfaces must originate from
   a tool lookup (`mb_get_component` for memory bank + board aggregation, or
   a `bv_*` tool that cross-checks the parsed board). These tools never
   fabricate — they return `{found: false, closest_matches: [...]}` for
   unknown refdes, and the system prompt instructs the agent to pick from
   `closest_matches` or ask the user. (2) Post-hoc sanitizer: every outbound
   agent `message` text is scanned for refdes-shaped tokens (regex
   `\b[A-Z]{1,3}\d{1,4}\b`) and, when a board is loaded, validated against
   `session.board.part_by_refdes`. Unknown matches are wrapped as
   `⟨?U999⟩` in the delivered text and logged server-side. Implementation:
   `api/agent/sanitize.py`.

## Stack

- **Backend:** Python 3.11+, FastAPI (~0.136), uvicorn, Pydantic v2,
  WebSocket (native), pdfplumber, pytest + pytest-asyncio
- **Agent:** `anthropic ~= 0.97.0` — tier-selectable at WS-open time:
  `deep` = Opus (`claude-opus-4-7`), `normal` = Sonnet, `fast` = Haiku
  (`claude-haiku-4-5`). The pipeline distributes Sonnet/Opus per sub-agent.
- **Frontend:** Vanilla HTML + CSS + JS (no build step, no bundler). All
  external assets come from permissively-licensed CDNs: D3.js v7
  (`d3js.org`), marked and DOMPurify (`cdn.jsdelivr.net`, both MIT) for
  safe Markdown rendering in the chat panel, and Inter + JetBrains Mono
  fonts (`fonts.googleapis.com`). No Tailwind, no Alpine, no component
  library. Any new CDN dependency must be permissively licensed and land
  in `web/index.html` with no transitive package-manager step.

## Commands

All tasks go through `make` (see `Makefile`):

```bash
make install   # create .venv and install deps (incl. [dev])
make run       # uvicorn api.main:app --reload on :8000
make test      # pytest tests/ -v -m "not slow" — fast subset (~1 min)
make test-all  # pytest tests/ -v — full suite incl. slow accuracy gates (7+ min)
make lint      # ruff check api/ tests/
make format    # ruff format api/ tests/
make clean     # drop __pycache__, .pytest_cache, .ruff_cache, egg-info
```

Single test / subset:

```bash
.venv/bin/pytest tests/board/test_test_link_parser.py -v
.venv/bin/pytest tests/agent/test_sanitize.py::test_wraps_unknown_refdes -v
.venv/bin/pytest -k "validator and not slow"
```

The API key is loaded from `.env` (copy `.env.example`). Tests do not require
`ANTHROPIC_API_KEY` — `api/config.py` defaults it to empty and only the
runtime code paths raise if it's missing.

**Test markers.** `make test` runs `-m "not slow"`. Any test that hits the
Anthropic API, ingests a real schematic, or acts as an accuracy gate must be
tagged `@pytest.mark.slow` so it only runs in `make test-all`. New tests
default to fast (no marker) and should stay under a second.

Bootstrapping Managed Agents (one-off, before the first `/ws/diagnostic`
session in `managed` mode):

```bash
.venv/bin/python scripts/bootstrap_managed_agent.py
# Creates the environment + 3 tier-scoped agents, writes managed_ids.json
# (gitignored). Re-runnable / idempotent.
```

## Layout

```
api/
  main.py            FastAPI app: /health, /ws/diagnostic/{slug}, mounts
                     web/ static, includes pipeline + board + profile routers
  config.py          Pydantic-settings Settings loaded from .env (cached)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Junkz3/wrench-board](https://github.com/Junkz3/wrench-board) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
