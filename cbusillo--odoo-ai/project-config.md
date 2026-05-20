---
trigger: always_on
description: This repo is retired for live Odoo runtime work. Keep it available for archival
---

# AGENTS.md — Retired Odoo Runtime Guide

This repo is retired for live Odoo runtime work. Keep it available for archival
reference only, and use `odoo-devkit`, tenant repos, and generated Odoo
workspaces for current runtime/build/deploy work unless the user explicitly asks
to inspect retired `odoo-ai` history.

---

## Historical Codex CLI Operating Guide

Treat this file as the launch checklist for every Codex session. Skim it, open
the linked docs, then keep prompts lean.

## Start Here

- Use the documentation table of contents (`docs/README.md`) to grab handles
  instead of copying long excerpts.
- Platform entrypoint: `uv run platform` for up/down/init/restore/build/ship/
  gate workflows (docs `docs/tooling/platform-cli.md`; implementation
  `tools/platform/cli.py`).
- Before changing code, open the matching style page (`docs/style/python.md`, `docs/style/javascript.md`,
  `docs/style/testing.md`).
- Clarify your role expectations with @docs/roles.md (analyst, engineer,
  tester, reviewer, maintainer).

## Project Snapshot

- Custom addons live under `./addons/`; host `./` maps to container `/volumes/`.
- We target Odoo 19 Enterprise. Never edit generated GraphQL artifacts
  (`addons/shopify_sync/services/shopify/gql/*`,
  `addons/shopify_sync/graphql/schema/*`).
- Always go through `uv run ...`; the Odoo environment must bootstrap every
  command (tests, scripts, shell helpers).
- Never call the system Python directly; use `uv run python ...` (or the
  scripted helpers) so the managed env stays in sync.
- Common helper entry points are defined in `[project.scripts]` inside
  `pyproject.toml` (examples: `test`, `platform`). Prefer them over ad-hoc
  commands and suggest additions when a useful script is missing.
- GPT service users seed automatically during restores when `.env` defines
  `ODOO_KEY`; see `docs/tooling/gpt-service-user.md` for provisioning details
  and API key usage.
- When you need multi-line scratch code, save it under `tmp/scripts/` and run
  `uv run python tmp/scripts/<name>.py` so the `uv run` sandbox bypass applies
  and you can iterate without heredocs.

## Version Guardrails (Odoo 19 + Owl 2)

- Views: use `<list>` roots, not `<tree>`.
- Views: use `invisible`/`readonly`/`required` and `column_invisible`; avoid
  legacy `attrs`/`states`.
- Frontend: native ESM only (`@web/...`, `@odoo/...`); no `odoo.define`.
- Frontend: do not add `/** @odoo-module */` in new files.

## Operating Guardrails

- Zero-warning + full-test acceptance gate: follow
  `docs/policies/acceptance-gate.md` and gate with `uv run test run --json`.
- Respect `docs/policies/coding-standards.md` and
  `docs/policies/doc-style.md` for naming, formatting, and docs-as-code.
- Naming guardrail: avoid abbreviations and 1–2 letter locals (e.g., `idx`,
  `cfg`, `tmp`, `obj`, `val`, `res`, `ctx`). Allow only explicit, well-known
  tokens (`id`, `db`, `api`, `orm`, `env`, `io`, `url`, `ui`, `ux`, `ip`,
  `http`, `json`, `xml`, `sql`) and math-only contexts.
- Update relevant docs in the same change when behavior shifts; link handles
  rather than pasting large snippets.
- Preserve history (`git mv`, minimal diffs) and avoid destructive git actions
  unless the operator explicitly directs them.
- Fix root causes, not symptoms: do not introduce workaround-only changes,
  temporary fallbacks, or bypasses unless the operator explicitly asks for a
  time-boxed mitigation.
- If production/runtime behavior is broken, pause and diagnose first. Before
  changing config/code, summarize the root cause hypothesis, validation steps,
  and intended permanent fix.
- Prefer fail-closed over silent degradation: if the right fix is blocked
  (credentials, infrastructure, missing access), stop and report the blocker
  clearly instead of masking it with alternate behavior.
- Never commit secret or operator-local env overrides (for example `.env`,
  `.platform/env/*.env`, `platform/secrets.toml`). Tracked
  templates/defaults like `.env.example` and `platform/config/base.env` are
  intended to be committed and must stay non-secret.
- For non-trivial work, prefer small checkpoint commits after each validated
  logical slice. Use those checkpoints as the base for review work so isolated
  follow-up fixes can be merged or `cherry-pick`-ed instead of manually
  re-applied. Keep commits coherent rather than per-turn, do not amend unless
  the operator explicitly asks, and fall back to manual porting when the
  checkout is too dirty or the review diff overlaps unrelated changes.
- Keep branch/worktree hygiene per @docs/roles.md: remove Code-created
  branches and temporary worktrees once merged or abandoned, and prune stale
  refs/worktrees as you go.

## Workflow Loop

- The working loop (plan → patch → inspect → targeted tests → iterate → gate)
  is spelled out in `docs/workflows/codex-workflow.md`.
- Use `docs/TESTING.md` to scope and shard tests via JSON summaries.
- Open the matching workflow playbook before deeper work:
  `docs/workflows/refactor.md`, `docs/workflows/debugging.md`,
  `docs/workflows/multi-project.md`, and `docs/workflows/prod-deploy.md`.

## Proactive Improvements

- Proactively suggest small environment or tooling improvements when you notice

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cbusillo/odoo-ai](https://github.com/cbusillo/odoo-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
