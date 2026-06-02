---
trigger: always_on
description: This file provides project-level guidance to coding agents (Claude Code, OpenCode, Codex, Copilot CLI) working in this repository. Treat it as authoritative alongside `CONTRIBUTING.md`; on conflict, prefer this file's specifics.
---

# AGENTS.md

This file provides project-level guidance to coding agents (Claude Code, OpenCode, Codex, Copilot CLI) working in this repository. Treat it as authoritative alongside `CONTRIBUTING.md`; on conflict, prefer this file's specifics.

## Source of truth

The implementation plan is the canonical design document:

- `docs/plans/2026-05-13-001-feat-llamatui-v1-launcher-plan.md` — v1 architecture, security contract, the nine Implementation Units (1: scaffold, 2: daemon/IPC, 3: GGUF, 4: discovery, 5: launch/supervisor, 6: TUI shell, 7: right-pane tabs, 8: non-interactive CLI, 9: release scaffolding), and what is explicitly out of v1.
- `docs/plans/2026-05-18-001-feat-init-wizard-doctor-pull-plan.md` — v2 plan covering R48–R80: init wizard, doctor diagnostic, `pull` MVP, recommender, fetch contract, snapshot bundling.
- `docs/brainstorms/llamatui-requirements.md` — origin requirements (R1–R46) that the v1 plan traces to.
- `docs/brainstorms/2026-05-18-init-wizard-requirements.md` — origin requirements (R48–R80) for v2.
- `docs/spikes/2026-05-19-*.md` — pre-implementation spike findings that anchor v2's design (hf-hub injection, GH Releases asset contract, brew Linux bottle, VRAM overhead).
- `docs/architecture.md` — stable user-facing summary of what's actually in the binary.

Before any non-trivial change, identify which Implementation Unit it falls under. PR descriptions should cite the unit; commit subjects often use `feat(unit5):` / `fix(unit3):` style.

## TODO tracking

`TODO.md` at the repo root is the single index of outstanding work. Any time
you add a TODO somewhere — a `TODO(...)` / `FIXME` comment in code, an
unchecked `- [ ]` in a plan or doc, a `todo:` frontmatter field on a spike,
or a deferred follow-up surfaced during review — also add a one-line entry
in `TODO.md` that links back to the source location. When you complete a
TODO, strike it from both places in the same change. The goal is that
`TODO.md` alone tells you everything still open without grep-walking the
tree.

## Docs stay in sync with code

Docs and code ship together. After any change that alters user-visible
behavior, the CLI / IPC surface, configuration shape, install paths, exit
codes, dependencies, scope boundaries, or architecture, update the affected
docs in the **same change** (same commit, same PR). Treat a PR that ships
code without the matching doc update as incomplete.
Agents working on this app must always keep core docs in sync (README,
AGENTS.md, feature docs, CHANGELOG, usage docs, install docs, and adjacent
user/developer docs touched by the change).

Files to review for drift on every change — skip the ones a change doesn't
touch, but check before assuming:

- `README.md` — install, quickstart, screenshots, feature list, exit-code
  table when present.
- `AGENTS.md` (this file) — scope boundaries, exit-code table, CLI agent
  surface, `status` IPC fields, build/test/lint, common gotchas.
- `INSTALL.md` — installer paths, prerequisites, and installation flows
  across supported environments.
- feature docs (`docs/brainstorms/*requirements*.md`, `docs/plans/*.md`,
  and feature-focused sections in `README.md` / `docs/usage.md`) — keep
  feature scope, status, and user-facing behavior aligned with shipped code.
- `CHANGELOG.md` — noteworthy user-visible changes land an entry under
  `[Unreleased]` (or the active release section). Internal-only refactors
  can be omitted. Entries must be **short, human-scannable one-liners** —
  not every small change earns a bullet, and bullets must not carry
  implementation detail or noise. Bundle related small changes into a
  single entry where it reads better, and link to the PR / commit (e.g.
  `(#123)` or short SHA) for anyone who wants the full story.
- `CONTRIBUTING.md` — workflow / contribution rules when they shift.
- `SECURITY.md` — only when the threat model or hardening surface shifts.
- `docs/architecture.md` — when modules, the IPC shape, lifecycle states,
  or the data-flow diagram change.
- `docs/usage.md` — CLI subcommands, flags, JSON output shapes,
  configuration keys, keybindings.
- `docs/troubleshooting.md` — new failure modes / error messages that an
  end user might hit.
- `docs/plans/*.md` — tick the corresponding unit checkbox `[ ]` → `[x]`
  when the work lands; never invent retro-plans, but do keep checkboxes
  accurate.
- `config.example.yaml` — when a config key is added, removed, renamed,
  or its default changes.
- `Cargo.toml` — keywords / categories / description on any feature that
  changes the binary's positioning.
- `TODO.md` — per the section above.

If a change makes an existing doc statement wrong, fix or remove the
statement; don't leave the contradiction for the next reader. If you
introduce a new user-facing concept that none of the above docs cover yet,
pick the doc closest in scope and add a section there rather than spawning
a new file.

## Scope boundaries

The v1 contract — these are deliberate omissions, not gaps:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [llamastash/llamastash](https://github.com/llamastash/llamastash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
