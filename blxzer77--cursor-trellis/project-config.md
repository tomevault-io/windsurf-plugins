---
trigger: always_on
description: These instructions are for AI assistants working in this project.
---

<!-- CSTL:START -->
# Cursor-Trellis (cstl) Instructions

These instructions are for AI assistants working in this project.

This project is managed by cursor-trellis. The working knowledge you need lives under `.cstl/`:

- `.cstl/workflow.md` — development phases, when to create tasks, skill routing
- `.cstl/spec/` — package- and layer-scoped coding guidelines (read before writing code in a given layer)
- `.cstl/workspace/` — per-developer journals and session traces
- `.cstl/tasks/` — active and archived tasks (PRDs, research, jsonl context)

If a cstl command is available on Cursor (e.g. `cstl-finish-work`, `cstl-continue`), prefer it over manual steps.

## Command surface (what is user-invocable vs internal)

Only a handful of Trellis entry points are meant for **manual `/` invocation**. Everything else is an **internal auto-triggered skill** — the agent loads it via the skill matcher or workflow routing, not by being called directly. Do **not** manually invoke internal skills through the slash palette.

- **User-invocable (manual)**: `cstl-continue`, `cstl-finish-work` (and `cstl-start` when needed).
- **Internal auto-triggered (do NOT call manually)**: `cstl-brainstorm`, `cstl-before-dev`, `cstl-check`, `cstl-break-loop`, `cstl-update-spec`, `cstl-micro-grill`, `cstl-meta`, `cstl-spec-bootstrap`, `cstl-skill-creator`, `smart-search-cli`. These activate on their own when the workflow/skill matcher decides they fit.

## Web research routing (smart-search first)

For **any external / current / web fact**, run **`python ./.cstl/scripts/run_smart_search.py "<question>" --intent deep-research --json`** first. That script is the **only** Trellis web-research evidence entrypoint (it shells out to the `smart-search` CLI). Do not guess paths under package source trees or sibling repos. Platform built-in web tools (Cursor `WebSearch` / `WebFetch`, or native web tools elsewhere) are **downgrade-only fallbacks**, used solely when smart-search is unavailable (`doctor` not ok, status `not_configured` / `failed`, or search timeout). Do not reach for built-in web search while smart-search is healthy. On Cursor, `smart-search-cli` is an **internal workflow skill name** only (not shipped under `.cursor/skills/`); follow `.cstl/spec/guides/retrieval-daily-guide.md` and `.cursor/rules/retrieval-routing.mdc` for the executable contract.

**External-knowledge gate:** If the answer would be wrong because the **world or a third-party API moved** and that matters → use smart-search (cheap `docs` / `broad-search` when enough; `deep-research` when multi-source). If truth lives only in this workspace → do not default to web. When unsure, prefer a cheap probe over guessing. See retrieval-daily-guide § External-knowledge gate.

Managed by cursor-trellis. Edits outside this block are preserved; edits inside may be overwritten by a future `cstl update`.

<!-- CSTL:END -->

## Mindfold harness (maintainers)

The Trellis CLI source repo often sits inside the **D:\MyHarness** harness: the harness root holds workspace-level `.cstl/` (tasks, spec, workflow) and is **not** a git repository. Run `git`, `pnpm`, and CLI validation from **this** directory (`Trellis/`). See `D:\MyHarness\AGENTS.md` for the three-repo layout (`Trellis/`, `smartsearch-private/`, `riverfjs-skills/`).

**Git remotes (local policy):** This checkout uses **only** the `private` remote (`git@github.com:blxzer77/cursor-trellis.git`). Do **not** add or push to `origin` / `mindfold-ai/Trellis`. Use `git push` (default remote is `private`) or `git push private <branch>`. Do not run `git push origin`.

**Branch policy (mandatory):** **`main` is integration/release only — never develop on `main`.** Before any durable edit, create or checkout a short-lived branch (`feat/…`, `fix/…`, `chore/…`). Do not commit feature work directly to `main`. Harness-wide rule: `D:\MyHarness\.cursor\rules\feature-branch-policy.mdc`.

---

# Trellis — AI Agent Codebase Guide (Cursor-only fork)

> Operational guide for AI agents editing this repository.
> This fork targets **Cursor** only (`--cursor`, optional `--cursor2plus`).

## 1. What Trellis Is

Trellis is a **team AI coding harness** — it turns monolithic `AGENTS.md` / `.cursorrules` into a progressive wiki of specs, tasks, workflows, and journals that agents load only when needed.

Published as npm package `@blxzer/cursor-trellis` with core SDK `@blxzer/cursor-trellis-core`. **Init and public docs are Cursor-only**; generated output is `.cursor/` (commands, rules, agents, hooks) plus `.cstl/`.

**Key concepts delivered to user projects**:
- `.cstl/spec/` — Team coding standards
- `.cstl/tasks/` — PRDs, context, status, acceptance criteria
- `.cstl/workspace/` — Developer journals and session continuity
- `.cstl/workflow.md` — Shared lifecycle: plan, build, check, finish, learn
- Cursor adapter — Generated `.cursor/` tree

---

## 2. Monorepo Architecture

```
Trellis/
  packages/
    core/              # @blxzer/cursor-trellis-core - domain primitives
    cli/               # @blxzer/cursor-trellis - CLI tool
  drafts/
  assets/
  .cstl/            # Self-dogfooding Trellis workspace
  .cursor/
  package.json
  pnpm-workspace.yaml
```

**Package manager**: pnpm 10.32.1 (monorepo workspaces)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blxzer77/cursor-trellis](https://github.com/blxzer77/cursor-trellis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
