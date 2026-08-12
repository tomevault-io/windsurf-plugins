---
trigger: always_on
description: Short entrypoint for Cursor / Codex / cloud agents. Prefer this over long essays.
---

# AGENTS.md — Boxlore agent contract

Short entrypoint for Cursor / Codex / cloud agents. Prefer this over long essays.

## Non-negotiables

- Read [`ARCHITECTURE.md`](ARCHITECTURE.md) + the touched module `README.md` before editing; **ARCHITECTURE wins** on conflicts.
- Before editing `scripts/sync/` (catalog pipeline): read [`scripts/README.md`](scripts/README.md) **and** the **Catalog sync** section below. Sync **does not** run on GitHub Actions — only on the Netcup VPS. A `git push` alone does **not** update the live runner.
- No feature→feature deps/imports; no PostHog in features (use `:core:analytics`); no Hilt/Koin/MockK.
- Never break identity/storage contracts (`applicationId`, DataStore `user_preferences`, Room names, `rss:` IDs, single `PlaybackRepository`, smart-queue refill ownership). See ARCHITECTURE identity table.
- Update the touched module README in the same change (template: [`docs/MODULE_README_TEMPLATE.md`](docs/MODULE_README_TEMPLATE.md)).
- Extend JVM `src/test` for touched logic; **bug fix ⇒ regression test** (same failure mode app-wide when shared). No Compose `androidTest` / emulator CI.
- Commit / push / open a PR **only when the user asks**. Conventional Commits titles.
- Every PR needs **exactly one** user-impact label (`user-impact-high|medium|low` or `no-user-impact`); optional `backend-change`. Changelog / README upcoming workflows depend on these — see [`.cursor/rules/pr-impact-labels.mdc`](.cursor/rules/pr-impact-labels.mdc).
- Merge when required checks are green (squash). Required checks: **`testDebugUnitTest`** + **`coderabbit-threads-resolved`**. SonarCloud / Gitleaks / CodeRabbit apps still run on PRs (fix Sonar issues). Unit suite cancels prior in-progress runs on new commits; `[skip unit]` / `[skip changelog]` only when appropriate. No merge queue / `merge-ci`.
- CodeRabbit (mandatory for agents):
  - Address every CodeRabbit finding and mark **every** CodeRabbit review thread **Resolved** before merge. Do not rely on the bare `CodeRabbit` status (that only means the review job finished). The hard gate is **`coderabbit-threads-resolved`**.
  - If the PR review decision is **`CHANGES_REQUESTED`** (CodeRabbit or anyone with write access): **stop**. Do **not** dismiss the review, do **not** force-merge / queue merge. Tell the user the PR is blocked on requested changes and ask them to merge (or dismiss) manually.
- SonarCloud: **0 new-code issues** on the PR (App quality gate). Fix Sonar findings; do not treat a missing Sonar ruleset requirement as permission to ignore them.
- Never commit secrets (`local.properties`, `.env`, keystores, `google-services.json`).
- Do **not** hand-edit `CHANGELOG.md` or README Upcoming / What's New regions (`<!-- release-upcoming:* -->` / `<!-- release-whats-new:* -->`) — `changelog-on-merge` owns those. Hand-edits are OK only for intentional release-note rewrites with matching script contracts.
- **boxlore-only:** do not change other `boxcreate` repos or org-wide bot settings unless asked. Keep proxy/backend internals out of public Android PR text.
- Product name in user-facing copy is **boxlore** (all lowercase), not “Boxlore” / “BoxLore”.
- Cards / panels: solid Material 3 surfaces only — no glassmorphism / translucent card backgrounds.

## Source of truth (priority order)

1. Latest user message (explicit overrides win)
2. This file + [`.cursor/rules/*.mdc`](.cursor/rules/)
3. [`ARCHITECTURE.md`](ARCHITECTURE.md)
4. Touched module `README.md`
5. [`docs/TESTING.md`](docs/TESTING.md) and [`.github/PULL_REQUEST_TEMPLATE.md`](.github/PULL_REQUEST_TEMPLATE.md)

## Where to look

| Topic | Doc |
| :--- | :--- |
| Module graph, DI, identity | [`ARCHITECTURE.md`](ARCHITECTURE.md) |
| Unit / Kover / Konsist / CI | [`docs/TESTING.md`](docs/TESTING.md) |
| Catalog sync pipeline (VPS, not GHA) | [`scripts/README.md`](scripts/README.md) |
| Always-on agent rules | [`.cursor/rules/`](.cursor/rules/) |
| PR body / merge checklist | [`.github/PULL_REQUEST_TEMPLATE.md`](.github/PULL_REQUEST_TEMPLATE.md) |
| Impact labels + merge gate | [`.cursor/rules/pr-impact-labels.mdc`](.cursor/rules/pr-impact-labels.mdc) |

## Catalog sync (VPS — not GitHub Actions)

**Hard stop before editing `scripts/sync/`.** Full detail: [`scripts/README.md`](scripts/README.md).

### Sync does not run on GitHub

- The old GHA workflow **`sync-pi-data` is sunset / removed**.
- There is **no** GitHub cron for charts, PI import, episode sync, or vectorization.
- Do **not** re-add a GitHub sync workflow unless the user explicitly asks.
- Do **not** assume `git push` to `master` updates the live pipeline by itself.

### Sync runs on the Netcup VPS

| What | Where |
| :--- | :--- |
| Live runner root | `/opt/boxlore-sync/` |
| **Code the cron executes** | `/opt/boxlore-sync/repo/scripts/sync/` (`run-sync.sh` → `cd $REPO`) |
| Orchestrator | `/opt/boxlore-sync/run-sync.sh` (systemd timers; panel install from `netcup-panel`) |
| Secrets / budgets | `/opt/boxlore-sync/.env` (never commit) |
| Run logs | `/opt/boxlore-sync/logs/runs/` |
| Local Turso / Qdrant | `/opt/boxlore-stack` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [boxcreate/boxlore](https://github.com/boxcreate/boxlore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
