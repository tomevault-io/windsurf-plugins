---
trigger: always_on
description: Canonical guidance for AI assistants (and humans) working in this repo, following the
---

# AGENTS.md — assaio

Canonical guidance for AI assistants (and humans) working in this repo, following the
[AGENTS.md](https://agents.md) convention. It overrides default assumptions. It loads into every
session and every subagent, so it stays short: a rule in one or two lines, the reasoning where it
lives — `CONTRIBUTING.md` (authoritative, shared with the community), `RELEASING.md`, `docs/adr/`,
`docs/corrections.md`. Path-scoped detail is in `.claude/rules/`; the harness in `.claude/README.md`.
Claude Code reads this file via the `@AGENTS.md` import in `CLAUDE.md`.

Roadmap: `ROADMAP.md` · Architecture decisions: `docs/adr/` · Documentation map: `docs/README.md`

## What this is

`assaio` measures how an organization uses AI coding tools and whether it is worth it:
cost/tokens, how much AI-written code reaches production, quality/bug impact, DevEx.
This repository ships one binary, `assaio-agent`: an offline-first CLI (Go, embedded
SQLite) that reads the local session logs of Claude Code, Codex CLI, Gemini CLI, GitHub
Copilot CLI, Cline, and Antigravity CLI (activity only — its format publishes no token counter,
so every cost figure withholds for it) and turns them into reports (`report`, `effectiveness`,
`reprice`), diagnostics (`analyze`, `check`, `doctor`, `status`), local session→commit
evidence (`evidence`), and the self-contained Assay HTML dashboard. Out-of-tree
exec plugins extend it in any language — parsers via `plugins:` (ADR 0003), metrics via
`metrics:` (ADR 0004), rules gating `check` via `rules:` (ADR 0005). A team-server MVP
(`serve` + `sync`) pools a team's usage on self-hosted infrastructure; the deeper org-analytics
server (git/issue-tracker correlation for survival/bug/quality) is future roadmap — see `ROADMAP.md`.

## Hard rules (long form: `CONTRIBUTING.md`)

- One file, one responsibility, ~200 lines. One metric = one file in `internal/analyze/`; one
  data source = one package in `internal/parser/`. Small functions, names that say what.
- Comments carry what code cannot — a contract, an invariant, a unit, why a wrong-looking thing
  is right — and take the space that needs. Never narration, attribution, dates or history.
- Program to interfaces; `internal/` never imports `plugin/` or `ee/` (depguard). No speculative
  abstraction, no dead code, no TODO dumps. Everything in code is English.
- Tests: table-driven, stdlib only, golden files from real captures (`-update`), a native
  `FuzzParse` for every parser.
- The gate: `make fmt` (no diff), `make lint`, `make test`, `make fuzz` on a parser change,
  `make vuln`; `.golangci.yml` is the canon. File size and complexity are review norms, not lint
  gates (ADR 0002) — never turn them into gates.
- Every `.md` under `docs/` is published or excused in `internal/docs/guides.go`; `make docs`
  regenerates `docs/reference.json` and the site, and `make test` fails on drift.

## Honesty rules (product-critical)

- Every domain fact carries its provenance and confidence; every figure states its measurement
  layer (activity / output / outcome / impact — ADR 0013). Lines are output, never "AI helped".
- Absence is never zero: an unsupported source, a failed read, a missing price render `—` and
  widen the unexplained share. Denominators declare their scope.
- Attribution and effectiveness claims ship with their error bars; prefer "directional" over a
  precise-looking wrong number. Bug density on AI lines compares only against age-matched code.
- Never an individual performance metric or leaderboard; pseudonymized is the default; prompts,
  responses, source code and diffs are never collected.

## Protected contracts (changed deliberately, on both sides at once)

- A shipped migration is immutable in name and content; every schema change is a new
  `internal/store/migrations/000N_*.sql` with its digest added to the test (`RELEASING.md`).
- What freezes at v1.0 — exec plugin protocols, the observation envelope, signal ids, the
  recommendation record, the sync protocol, machine-readable outputs — is listed once in
  `docs/compatibility.md`; pre-1.0 a break is announced under **Breaking**.
- `B` ids in `BACKLOG.md` are never reused; `CHANGELOG.md` keeps the seven Keep-a-Changelog
  headings; published tags are immutable.
- The maintainer's real store, `~/.local/share/assaio/assaio.db`, is never opened by a test or
  an agent run: every local run sets `XDG_DATA_HOME=$(mktemp -d)`.
- `site/` names no version, loads nothing at render time, and publishes on every push to `main`.

## Git & commits

- Do not commit unless asked; commits are made by hand after a milestone. Stage explicit paths
  only — never `git add .`, `-A` or `commit -a`: another session may hold work in this tree.
- Conventional Commits with a concrete subject, a short body only when the why is not obvious,
  `Signed-off-by` (DCO), one commit per PR, `main` protected (PR + green CI + review).
- Never credit an AI assistant as an author, anywhere; the hook and the `dco` job reject it.

## How work happens here

- Skills: `/work` → `/plan` → `/build` → `/gate` → `/review` → `/ship`; `/auto` runs the chain
  unattended; `/research` feeds the roadmap; `/add-source`, `/debug`, `/refactor`, `/docs`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [assaio/assaio](https://github.com/assaio/assaio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
