---
trigger: always_on
description: Orientation for an AI coding agent (Claude Code, Codex, Cursor, or similar) working in this repository. If you're a human, [`README.md`](./README.md) and [`docs/`](./docs) are the better starting point — this file is dense and reference-oriented on purpose.
---

# AGENTS.md

Orientation for an AI coding agent (Claude Code, Codex, Cursor, or similar) working in this repository. If you're a human, [`README.md`](./README.md) and [`docs/`](./docs) are the better starting point — this file is dense and reference-oriented on purpose.

This follows the [AGENTS.md](https://agents.md) standard — read natively by Codex and other AGENTS.md-aware tools. **Claude Code does not read AGENTS.md on its own** (confirmed against Anthropic's own docs — it only ever auto-loads `CLAUDE.md`), so `CLAUDE.md` in this repo's root just imports this file via `@AGENTS.md`. That's also exactly the gap `src/reuse.js`'s `injectAgentsMd()` needs to account for when writing to a *target* project — see that file's own doc comment.

## What this tool is

Mycelium is a local-first Context Lifecycle TUI for AI coding-agent sessions. It manages context produced by AI collaboration through four stages — **Capture → Organize → Learn → Reuse** — implemented as a terminal UI (`neo-blessed`) plus a CLI, both reading/writing a plain-file store under `~/.mycelium/`.

## Why it exists

AI coding sessions (Claude Code, Codex, Kiro, OpenCode, ...) accumulate fast and lose context across three axes:
- **Model boundaries** — switching agents mid-task loses everything the previous agent knew (see Handoff below).
- **Time** — a session from three weeks ago is unfindable without search/organization.
- **Space** — nothing links related sessions across a project's lifetime unless something builds that structure.

Mycelium's answer: capture every session losslessly into a neutral schema, organize it (LLM-assisted, human-confirmed) into folders, learn from finished sessions (auto-summarize, extract project knowledge), and reuse that knowledge — injected into `AGENTS.md` for the next agent, or composed into a handoff prompt when switching agents mid-task. See [`docs/features.md`](./docs/features.md) for the full capability catalog and [`docs/architecture.md`](./docs/architecture.md) for design principles (local-only, model-agnostic, human-first, minimal dependencies).

## Repository layout

```
src/
  cli.js              CLI entry point (25+ subcommands) — see docs/cli.md
  paths.js            ~/.mycelium/* path constants; HOME resolved from MYCELIUM_HOME once at import time
  schema.js           the neutral session schema every adapter normalizes into
  scanner.js          Capture: import from each adapter, raw/ file CRUD
  organize.js         barrel — see src/organize/{folders,classify,lineage}.js
  daemon.js           barrel — see src/daemon/{cycles,process}.js
  learn.js            auto-tagging (title/summary/tags/decisions/todos) via an LLM call
  insight.js          digests + per-folder KNOWLEDGE.md generation
  reuse.js            KNOWLEDGE.md → AGENTS.md injection (ancestor-path inheritance)
  handoff.js          cross-agent handoff prompt composition
  backlog.js          user-written intent notes (kind: 'backlog'), opened as a seeded handoff
  split.js            LLM-suggested session splitting
  index-db.js         sqlite (FTS5) index — derived, rebuildable from raw/
  config.js           config.json read/write (locale, excluded ids, etc.)
  llm.js              headless LLM calls via the user's own claude/codex CLI subscription
  agents.js           derives binFor/resumeArgsFor from the adapter registry
  adapters/           one file per agent CLI (claude-code.js, codex.js, kiro.js) + index.js registry
  tui/                the neo-blessed interface — app.js (shell), views/ (sessions.js, calendar.js),
                       widgets/ (pickers.js, viewers.js), data.js (thin read layer over scanner+index-db),
                       tutorial.js + tutorial-data.js + tutorial-mock-llm.js + personas.js (first-run tour / `mycelium demo`)
docs/                 detailed guides (linked from README.md's "Learn More")
test/                 node:test suite — see "Tests" below
.github/              CI/CD workflows, issue/PR templates
```

### Architecture pattern: barrel modules, not deep layers

`organize.js` and `daemon.js` are **barrels**: `export * from './organize/*.js'` / `export * from './daemon/*.js'`. The implementation lives in sibling files split by responsibility (`organize/folders.js` = folder CRUD, `organize/classify.js` = LLM classification workflow, `organize/lineage.js` = manual mutation + merge/split/continuation; `daemon/cycles.js` = cadence/policy, `daemon/process.js` = OS process lifecycle), but every existing importer keeps using `'../organize.js'` / `'./daemon.js'` unchanged. This mirrors k9s's per-resource-type accessors behind a shared interface (`internal/dao`'s `Accessor`/`AccessorFor`) — the same shape as this repo's own `src/adapters/index.js` (`ADAPTERS` array + `getAdapter(source)`). **If you're adding new functionality to `organize.js`/`daemon.js`, add it to the right sibling file, not to the barrel.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [krapie/mycelium](https://github.com/krapie/mycelium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
