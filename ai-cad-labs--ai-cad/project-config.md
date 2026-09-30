---
trigger: always_on
description: Agent-first onboarding for this repository, for any coding assistant:
---

# AGENTS.md · AI-CAD

Agent-first onboarding for this repository, for any coding assistant:
Claude Code, Codex, Cursor, opencode, Gemini CLI, and others.
If you are an AI agent (or a human) working in this codebase, read this first.
Then match your next read to your mode:
**developing or changing the system** means also reading [`README.md`](README.md)
before touching anything
(it carries the project-level context this file deliberately does not duplicate:
the guiding design principles, what a run produces, the run gallery);
**merely running design runs** (execution mode) does not require the README.
Many assistants read `AGENTS.md` natively as the project-rules entry point;
Claude Code and Gemini CLI arrive here
through the one-line imports in `CLAUDE.md` and `GEMINI.md`.

AI-CAD is a multi-agent system that designs manufacturable parts and assemblies:
LLM agents generate CadQuery (parametric Python CAD) code,
render it to engineering views,
evaluate it against design-for-manufacturing / design-for-assembly rules,
and iteratively repair the geometry.
Orchestration lives in agent instructions
plus a deterministic, bash-callable Python tool layer,
so the product runs on Claude Code today
and is symlink-portable to other harnesses.

## Read-first map (in order, for any change)

1. **This file, then `README.md` (required in development mode)**:
   this file is the tactical layer (conventions, runbook, fences);
   the README is the project-level layer
   (guiding design principles, artifact expectations, the gallery)
   that any change to the system must stay aligned with.
   The two are deliberately non-overlapping: read both, duplicate neither.
2. **`.shared/agents/<role>/instructions.md`**: the 9 agent roles (the operating brains).
3. **`.shared/skills/`**: the knowledge layer
   (cadquery-cookbook, dfm-rules, run-reflection,
   cadquery-anti-hallucination, and the two reserved stubs).
4. **Design system (frontend work)**: `frontend/.interface-design/system.md` +
   `frontend/src/index.css` (Tailwind v4 token layer).
5. **[`docs/dev/local-setup.md`](docs/dev/local-setup.md)**: developer environment setup,
   including local clones of the upstream CadQuery repos for deep reference.
6. **[`docs/README.md`](docs/README.md)**: the docs index
   (the glossary, the architecture decision records, and navigation pointers).

## Repo structure

```
ai-cad/
├── AGENTS.md                              # this file: the cross-assistant agent rules
├── CLAUDE.md / GEMINI.md                  # one-line imports of AGENTS.md
├── README.md                              # public face
├── .shared/                               # CANONICAL content (edit here, never via symlinks)
│   ├── agents/<role>/instructions.md      # 9 roles
│   ├── skills/                            # cadquery-cookbook, anti-hallucination, dfm-rules, run-reflection, + stubs
│   └── tools/                             # bash-callable Python tools (python -m tools.<name>)
├── tools -> .shared/tools                 # import-path symlink
├── .claude/  .opencode/                   # per-harness config (symlinks into .shared/)
├── rules/                                 # DFM/DFA rulebooks (JSON) + .proposed siblings + lifecycle log
├── reference/                             # 5 distilled CadQuery reference docs (README, API_SURFACE, COOKBOOK, 2 cheatsheets)
├── projects/<name>/                       # RUN OUTPUT (untracked): goals, design_log, assembly/<part>/…
├── frontend/                              # read-only run dashboard (Vite + React + TS + Tailwind v4 + shadcn)
└── tests/                                 # uv run pytest
```

## Sibling repositories

- [`ai-cad-labs/ai-cad-example-projects`](https://github.com/ai-cad-labs/ai-cad-example-projects):
  the run gallery of real, unedited design runs from this system (both harness generations).
  Consult it to see what healthy run output looks like before changing pipeline behavior.
- [`ai-cad-labs/ai-cad-pydanticai`](https://github.com/ai-cad-labs/ai-cad-pydanticai):
  the predecessor PydanticAI implementation (frozen; research value).
  Documents the architecture this harness grew out of.

## Runbook

**Environment**: `uv sync`;
install system cairo
(macOS: `brew install cairo`, and the renderer self-sets the `DYLD` path;
Debian/Ubuntu: `apt-get install libcairo2`).
No `.env` and no API keys exist by design:
the `claude` CLI carries its own auth.
DFMA/vision evaluation runs on Claude via the `claude` CLI:
optional env `CAD_EVALUATOR_MODEL` pins a model
(a large-context variant is recommended for long pipelines),
else the CLI default;
`CAD_EVALUATOR_TIMEOUT_S` defaults to 300;
`CAD_EVALUATOR_CONCURRENCY` caps concurrent evaluations in a sweep (defaults to 4).
(`.claude/settings.json` ships an optional `CAD_EVALUATOR_MODEL` pin
you can override or remove.)

**Launch a design run**
(headless, detached;
dispatch the orchestrator SYNCHRONOUSLY within the turn,
because headless `-p` exits at turn end
and would orphan-kill an in-process background child):

```bash
claude -p "<entry prompt: create projects/<name>/ with goals.md, dispatch the orchestrator to
run to completion within THIS turn, write smoke_report.md, THEN synthesize run_reflection.md

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ai-cad-labs/ai-cad](https://github.com/ai-cad-labs/ai-cad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
