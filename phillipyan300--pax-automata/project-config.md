---
trigger: always_on
description: This file defines the **permanent operating rules** for Claude Code in this repo. It is the "Rules of the Land": immutable conventions. It is **not** a scratchpad or status log. Do not store task lists or progress here—use `docs/project-tracker.md` for that.
---

# Pax-Automata — Claude Code Constitution

This file defines the **permanent operating rules** for Claude Code in this repo. It is the "Rules of the Land": immutable conventions. It is **not** a scratchpad or status log. Do not store task lists or progress here—use `docs/project-tracker.md` for that.

---

## 1. Project Identity

- **Repo:** Pax-Automata — autonomous agent for **Pax Historia** (browser grand-strategy game).
- **Architecture:** Cognitive loop: **Perception (Spy)** → **War Room (local files)** → **Brain (LLM)** → **Execution (Hand)**.
- **Source of truth for product/design:** `docs/PRD.md`, `docs/diagrams/war-room-flow.mmd`, `docs/diagrams/war-room-flow.png`.

---

## 2. Tech Stack (Canonical)

| Layer        | Choice              | Notes |
|-------------|---------------------|--------|
| Browser automation | **Playwright** | Intercept `/api/simple-chat`, drive UI (action box, advisor box). |
| Runtime     | **Node.js + TypeScript** | Preferred for Playwright and tooling; use Python only if explicitly required. |
| LLM         | **Anthropic Claude API** (via SDK) | Reasoning and batch-of-actions generation. |
| Validation  | **Zod**              | Schema validation at all JSON boundaries. |
| TS execution | **tsx**             | Fast TypeScript execution (replaces ts-node). |
| War Room storage | JSON + Markdown + TXT in `war-room/` | See folder layout below. |

---

## 3. Folder Structure (Canonical Paths)

```
/
├── CLAUDE.md                 # This file — constitution only.
├── docs/
│   ├── PRD.md                # Product/technical spec (read first for features).
│   ├── project-tracker.md    # Status scratchpad: done vs pending (update here, not in CLAUDE.md).
│   ├── diagrams/
│   │   ├── war-room-flow.mmd # Mermaid sequence diagram.
│   │   └── war-room-flow.png # Rendered diagram.
│   └── logs/                 # Timestamped logs for refactors/debug sessions (e.g. 2026-02-07-perception-layer.md).
├── war-room/                 # Agent identity & memory (in-repo templates; runtime may copy or symlink).
│   ├── constitution.md       # Long-term goals (fixed).
│   ├── crisis_handbook.txt   # Tactical playbook.
│   ├── strategic_ledger.json # Active plans / long-term memory.
│   └── current_state.json    # Latest game state (written by Spy).
├── src/                      # Application code.
│   ├── spy/                  # Perception layer (Playwright intercept, state parsing).
│   ├── brain/                # Reasoning layer (context assembly, LLM calls, action generation).
│   ├── hand/                 # Execution layer (Playwright UI automation).
│   └── shared/               # Types, Zod schemas, utilities shared across modules.
└── scripts/                  # One-off or scaffold scripts (e.g. scaffold.sh).
```

**Search / references:** Prefer `docs/PRD.md`, `docs/diagrams/war-room-flow.mmd`; War Room files live under `war-room/`.

---

## 4. Naming Conventions

- **Files:** `kebab-case` for scripts and config; `snake_case` for War Room data files to match PRD (`current_state.json`, `strategic_ledger.json`).
- **Branches:** `feature/<short-name>`, `fix/<short-name>`, or `chore/<short-name>` (e.g. `feature/perception-layer`, `fix/ledger-schema`, `chore/deps`).
- **Commits (on feature branches):** Clear, present-tense messages; meaningful and scoped (checkpoints are fine, but avoid noisy micro-commits).

---

## 5. Pre-flight Rule

**Before creating any new file or module**, check this file (`CLAUDE.md`) to confirm the correct path, naming convention, and module boundary. If the file doesn't fit the canonical structure, stop and discuss.

---

## 6. Coding Standards

- **Functional-first:** No classes unless there's a clear reason (e.g. Playwright Page wrapper). Prefer plain functions and modules.
- **Zod at boundaries:** Every JSON file read/write and every external API response gets a Zod schema. Validate, don't assume.
- **Explicit return types** on all exported functions.
- **Barrel exports:** Each module directory (`spy/`, `brain/`, `hand/`, `shared/`) has an `index.ts` re-exporting its public API.
- **No `any`:** Use `unknown` + Zod parsing instead. TypeScript strict mode.
- **Imports:** Prefer relative imports within a module; use `../shared/` for cross-module shared code.

---

## 7. Architecture — Data-Flow Contracts

The cognitive loop passes data through well-defined boundaries:

```
Spy (Playwright intercept)
  → writes current_state.json (Zod-validated GameState)
  → emits "new-turn" event

Brain (context assembly)
  ← reads current_state.json, constitution.md, crisis_handbook.txt, strategic_ledger.json
  → calls Anthropic Claude API with assembled prompt
  → receives ActionBatch (Zod-validated)
  → updates strategic_ledger.json

Hand (Playwright UI driver)
  ← receives ActionBatch from Brain
  → types each action into the action box, submits
  → optionally queries advisor, returns advice to Brain
```

Each boundary has a Zod schema. Modules communicate through files (War Room) and function calls — no shared mutable state.

---

## 8. Workflow + Git Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phillipyan300/Pax-Automata](https://github.com/phillipyan300/Pax-Automata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
