---
trigger: always_on
description: Instructions for coding agents (Claude, Cursor, Grok, Codex, etc.) working in this repo.
---

# Agent notes — DevCake

Instructions for coding agents (Claude, Cursor, Grok, Codex, etc.) working in this repo.
These rules are **mandatory** unless the user explicitly overrides them for a task.

For setting up or operating a deployment, read the agent-neutral
[`skills/devcake-ops/SKILL.md`](skills/devcake-ops/SKILL.md). It explains the
product, configuration, diagnostics, and recovery workflow for an operator's helper.

## Security / product claims

The **product security contract** is [`docs/14-security.md`](docs/14-security.md).
Do not write docs, README, or PR copy that claims a stronger posture (multi-tenant
sandbox, secrets never leave the host under injection, hard-gated branch
protection) than that file. Do not treat staffing a different Dev Type for
REVIEW as a security control — the security-relevant second identity is the
**reviewer token** (app-only); REVIEW is always a pipeline stage. Design choices
(dedicated host, adult-operator prompt trust, warnings vs gates) are intentional.

## Admin SPA design system (mandatory)

Any change under `admin/spa/` that affects look, feel, copy, or interaction
**must follow [`admin/spa/DESIGN.md`](admin/spa/DESIGN.md)** — the decided
design guideline (identity, tokens, layout idioms, action hierarchy, dialogs,
copy voice, evidence loop). Read it before touching the SPA. Iron rules:

- Colors come from the `@theme` tokens in `admin/spa/src/index.css` — never raw
  hex or new color families in components. `accent-*` is the only brand accent.
- Scalar settings are `SettingRow`s; record lists are real tables styled like
  the Runs table; Fleet (`#/fleet/<section>`) and Settings (`#/settings/<section>`)
  each render one section per view (Limits carries the merged Limits + Traffic
  cards); Repositories, PMO, and Skill sources are draft-editing pages under
  the sidebar's Connections item — not Fleet/Settings sections.
- One primary action per header/card; secondary/rare/destructive actions go in
  a `MoreMenu` (⋯) with honest one-line consequence descriptions — but never a
  one-item menu when it's the element's only action.
- Native `window.confirm/prompt/alert` are banned — use `Modal.jsx` dialogs.
- Draft semantics are untouchable: config edits ride `useConfigDraft`; anything
  immediate is wrapped in `InstantZone`.

## Always Works™ (mandatory before done)

**"Should work" ≠ "does work."** Before marking any change complete, prove it with evidence you personally observed — not assumptions.

| Change type | Minimum proof |
|---|---|
| Docker / Bake / Compose | `docker buildx bake …` succeeds **and** `docker compose up -d` + healthchecks pass |
| App / API | **`./scripts/pytest_app.sh`** (always rebuilds `app-test` via Bake or Dockerfile `--target test`, then pytest), **or** `docker buildx bake app-test` then pytest in that image, **or** `PYTHONPATH=app` on Python **3.12** against the working tree; **and** bake/restart prod `app` when the run path changed |
| Admin SPA | `bake admin` **and** load UI / nginx-health |
| Dev harness / entrypoint | `bake images` (or affected target) **and** smoke CLI + import entrypoint |
| Docs-only | No runtime required; still re-read for accuracy |

**Stale `app-test` trap:** the `devcake/app-test` image **COPY**s `app/devcake` and `app/tests` at bake time. Re-running pytest on an old `devcake/app-test:latest` grades the last bake, not your working tree — a silent false green. Always rebake after `app/` edits, or use `PYTHONPATH=app` on 3.12, or `./scripts/pytest_app.sh` (always rebuilds first via `scripts/lib/bake_app_test.sh`: Docker Buildx bake when available, else `docker build -f app/Dockerfile --target test`). CI rebakes on every run; local agent loops often forget.

Never claim done from "build succeeded" alone when the user-facing path is run/up. Name anything still unproven.

## Engineering standards (mandatory for new work)

### TDD for new implementations

**New behavior is test-first.** Do not implement production code for a new feature, bug fix with a known reproduction, or new module until a failing test exists.

| Rule | Detail |
|---|---|
| **Red → green → (then ship)** | Write a failing test that names the behavior; write the minimum code to pass; only then refactor. |
| **Vertical slices** | One seam / one behavior at a time. Do **not** bulk-write a suite of imagined tests then implement everything. |
| **Agree the seam first** | Before the first test: state the public interface under test (function, port Protocol, HTTP path). Tests hit that seam only. |
| **No private tests** | Do not assert on private helpers, call counts of internal collaborators, or implementation structure. Prefer fakes at **port** seams (`ports/*`). |
| **Independent expected values** | Assertions use known literals / domain rules — not recomputing the same algorithm as production. |
| **Where tests live** | `app/tests/test_*.py`. Run with Python **3.12** (prod image). Prefer `./scripts/pytest_app.sh` (rebakes `app-test` then pytest), or `PYTHONPATH=app` locally on 3.12. |
| **When TDD does not apply** | Pure renames, docs-only, config/copy, or mechanical follow-the-existing-pattern refactors with no behavior change — still run existing tests (Always Works™). |

### SOLID (always)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flieber-inc/devcake](https://github.com/flieber-inc/devcake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
