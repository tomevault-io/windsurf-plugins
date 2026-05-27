---
trigger: always_on
description: This file is the **developer guide** for the solstone repository. Read it before writing code.
---

# solstone Developer Guide

This file is the **developer guide** for the solstone repository. Read it before writing code.

Audience:

- **Coders** (cwd = repo root, editing `solstone/observe/`, `solstone/think/`, `solstone/convey/`, `solstone/apps/`, `solstone/talent/`, `tests/`) — you're in the right place.
- **Cogitate talents** (cwd = `journal/`, running inside the live system) — your entry is `solstone/talent/journal/SKILL.md`, installed into `journal/.claude/skills/journal/` and `journal/.agents/skills/journal/`.
- **Operators** debugging a running system — see `docs/DOCTOR.md`.

For the journal-side runtime entry point, see `journal/AGENTS.md`.

`CLAUDE.md` and `GEMINI.md` at the repo root are symlinks to this file.

## 1. Start here

Read, in order, when you enter the repo for a coding task:

1. **This file through §8** — the invariants must be in working memory before your first edit.
2. **`solstone/think/sol_cli.py`** — the CLI entry point. Skim the `COMMANDS`, `ALIASES`, and `GROUPS` dicts. ~340 lines, scannable in one pass. You now know the whole top-level command surface.
3. **`solstone/think/top.py` (first ~100 lines)** — the interactive TUI. Ties callosum + supervisor + service status together in one vantage point. Good "oh, this is how it connects" moment.
4. **The area you're about to touch:**
   - User-visible feature or `sol call <app> <verb>` → `solstone/apps/<name>/call.py` + `solstone/apps/<name>/routes.py` + `solstone/apps/<name>/templates/`.
   - Think pipeline → `solstone/think/<module>.py` + its tests.
   - AI talent prompt or behavior → `solstone/talent/<name>.md` (+ optional `.py` post-hook).
   - Capture / observe → `solstone/observe/<module>.py`.
5. **Run `sol`** (no args) — prints current journal status + grouped command list. Orients you to live state.
6. **`make dev`** or **`make sandbox`** when you need a running stack to iterate against.

> If you cannot state in one sentence **which module owns the data your change touches**, stop and re-read §7 L2 (the domain ownership table). Writing to a domain from the wrong module is how we got the 14 layer violations the April 2026 audit catalogued.

## 2. Repo map

| Dir | Purpose | Go here when | Depth doc |
|-----|---------|--------------|-----------|
| `solstone/think/sol_cli.py` | CLI entry point — `COMMANDS` / `ALIASES` / `GROUPS` dicts | adding a top-level `sol <cmd>` | `docs/SOLCLI.md` |
| `solstone/observe/` | Multimodal capture — screen, audio, transcribe, describe, sense, transfer | capture-side bugs, new input modalities | `docs/OBSERVE.md` |
| `solstone/think/` | Post-processing core — cortex, talent, callosum, indexer, entities, facets, activities, scheduler, heartbeat, supervisor | anything downstream of capture; most coder work lives here | `docs/THINK.md`, `docs/CORTEX.md`, `docs/CALLOSUM.md` |
| `solstone/convey/` | Web app framework — app discovery, routing, bridge | layout / framework-level UI changes | `docs/CONVEY.md` |
| `solstone/apps/` | Convey apps — each self-contained (`call.py` Typer sub-app + `routes.py` + `templates/`) | adding a user-facing feature, a `sol call <app>` verb, a UI surface | `docs/APPS.md` (required reading before modifying `solstone/apps/`) |
| `solstone/talent/` | AI talent configs (markdown prompts + optional `.py` post-hooks) + `SKILL.md`s (journal, coder, partner, …) | defining or tuning a talent; adding a journal-side skill | `solstone/talent/journal/SKILL.md`, `docs/PROMPT_TEMPLATES.md` |
| `scripts/` | Repo maintenance scripts — `check_layer_hygiene.py` | tooling that guards the codebase; wired into `make ci` | (none) |
| `tests/` | Pytest suites + `tests/fixtures/journal/` mock journal | writing tests; debugging flakiness; `make dev` / `make sandbox` use fixtures as the journal | `docs/testing.md` |
| `docs/` | All longform documentation | reference lookups; never your first stop | §10 below |
| `journal/` | The live journal (user data). Git-ignored content; checked-in template (`AGENTS.md`, skills symlinks) | **rarely as a coder** — modify `solstone/think/`, `solstone/apps/`, or `solstone/talent/`, not journal data | `solstone/talent/journal/SKILL.md` |

Top-level dirs intentionally not in the table: `.venv/`, `scratch/`, `logs/`, `tmp/`, `observers/`, `routines/`, `skills/` — not active coder surfaces.

## 3. Mental model

**The pipeline:** `observe` (capture) → JSON transcripts in `journal/chronicle/YYYYMMDD/` → `think` (analyze) → SQLite index + derived artifacts → `convey` (web UI) and `sol call` CLIs.

**Think is the center.** observe feeds it raw material; convey + apps render its outputs; talent prompts + cortex run AI against it; indexer makes it searchable. A change in `solstone/think/` usually ripples outward.

**Key concepts, priority-ordered:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [solpbc/solstone-journal](https://github.com/solpbc/solstone-journal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
