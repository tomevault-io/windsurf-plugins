---
trigger: always_on
description: JellyRock is a Jellyfin client for Roku, written in **BrighterScript** (`.bs`, transpiled to `.brs`) with **Roku Scene Graph** (`.xml`) for the UI. The Jellyfin REST API is wrapped by an in-house client + persistent task pool — see [`source/api/CLAUDE.md`](source/api/CLAUDE.md) and [`docs/architecture/api.md`](docs/architecture/api.md)
---

# JellyRock — Agent Rules

JellyRock is a Jellyfin client for Roku, written in **BrighterScript** (`.bs`, transpiled to `.brs`) with **Roku Scene Graph** (`.xml`) for the UI. The Jellyfin REST API is wrapped by an in-house client + persistent task pool — see [`source/api/CLAUDE.md`](source/api/CLAUDE.md) and [`docs/architecture/api.md`](docs/architecture/api.md)

## ⚠️ Mandatory rules

1. DO NOT make stuff up or make assumptions
2. Ask clarifying questions when you are not sure about something
3. Focus on best practices, industry standards, easy long-term maintenance, no regressions, and world-class UX and DX
4. ALWAYS look for the best possible solution to a problem then provide the user with their best options
5. Iterate on a plan with the user until they approve it, and only then begin coding
6. After finishing a user-approved plan: run automated tests to verify; provide a manual test plan only for UI/runtime behavior tests don't cover, plus any expected debug-log output

## Agent rules

- **Run tests to verify fixes — don't commit based on reasoning alone.** Nothing auto-runs tests, so an agent is expected to run them. BS unit tests on Roku hardware — TDD (single spec, fastest): `npm run test:tdd`; broader: `npm run test:unit | test:integration | test:all`. BSC plugin / scripts changes (Vitest, no hardware needed): `npm run test:scripts`. Setup, credentials, debugger contention: [`docs/dev/unit-tests-tdd.md`](docs/dev/unit-tests-tdd.md).
- **When hardware isn't reachable, say so explicitly** — don't claim a fix was tested when only the build was verified
- **Cannot modify `CHANGELOG.md`** — CI-controlled
- **Don't compulsively re-run lint / build / format mid-work.** `npm run validate`, `lint:*`, `build:*`, `check-formatting`, and `format` are already run by pre-commit / pre-push hooks and by CI on every push (and most editors surface BSC diagnostics live as you type). So they aren't for routine "did my change compile" checks — but they're fair game when debugging a specific failure, when no hook has fired yet, or when your editor isn't surfacing diagnostics. **Test scripts are NOT covered** — `test:tdd` / `test:unit` / `test:scripts` aren't auto-run anywhere before commit, so running them as part of finishing work is the expected workflow, not a redundancy.
- **Capture cross-session agent guidance in `CLAUDE.md` (root or scoped), not in agent-private memory** — memory files are per-folder (worktrees / multiple JellyRock checkouts each get their own), aren't committed, and don't reach other contributors. Project rules belong in `CLAUDE.md` so everyone benefits. Auto-memory is disabled at the project level (`.claude/settings.json` → `autoMemoryEnabled: false`)
- **Don't reference `tasks/` paths in shared artifacts** — `tasks/` is gitignored; reviewers can't navigate there. Keep it out of commit messages, PR bodies, and shared docs
- **PR follow-ups land in a journal, not just the PR body** — when a PR explicitly defers something ("out of scope", "follow-up"), add an entry to the right journal (see Capture & state discipline below) and link it from the PR. Otherwise the deferral evaporates the moment the PR merges

## Capture & state discipline

The four-pillar journal system (see [`docs/architecture/system-shape.md`](docs/architecture/system-shape.md)) treats live project state as load-bearing. Three rules govern how agents interact with the journals:

- **Capture-discipline rule** — when committing a decision-shaped change (a choice that closes off alternatives, has a non-obvious rationale, or has a constraint worth re-evaluating), invoke `/log decision` in the same change set. **Raw markdown edits to [`docs/decisions.md`](docs/decisions.md), [`docs/progress.md`](docs/progress.md), or [`docs/signals-backlog.md`](docs/signals-backlog.md) are not the sanctioned path for agents** — use `/log` (capture) and `/done` (close) skills exclusively. Direct `Write` / `Edit` on those three files bypasses the diff-and-wait safety net and risks silent corruption of project state. *CI exception:* the post-merge [`.github/workflows/journal-sync.yml`](.github/workflows/journal-sync.yml) workflow is the sole non-skill writer to `progress.md`, performing the mechanical close-loop (move `## Currently running` → `## Recently shipped`, bump `last-updated:`) via [`scripts/journal-sync.js`](scripts/journal-sync.js). Judgment-bearing entries (decisions, tech-debt, followups) still flow through the user-driven `/pr` → `/log` path.
- **Followup-discipline rule** — when deferring work in a PR ("out of scope", "follow-up", "TODO later"), pick the right journal:
  - Internal debt with a slug + severity (refactor candidate, design intent worth preserving) → invoke [`/tech-debt-scan`](.claude/skills/tech-debt-scan/SKILL.md) (writes to [`docs/architecture/tech-debt.md`](docs/architecture/tech-debt.md))
  - Generic deferred work without a debt classification → invoke `/log followup "<text>" --area=<name>` (writes to [`docs/progress.md`](docs/progress.md))

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jellyrock/jellyrock](https://github.com/jellyrock/jellyrock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
