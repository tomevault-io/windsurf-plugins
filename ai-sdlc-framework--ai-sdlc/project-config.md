---
trigger: always_on
description: - **Always rebase** feature branches onto main; never merge main in.
---

# AI-SDLC Project Instructions

## Git Flow

- **Always rebase** feature branches onto main; never merge main in.
- Update branch: `git fetch origin && git rebase origin/main`, then `git push --force-with-lease`.
- Never `gh api pulls/N/update-branch` with merge method. Keep linear history.
- `/ai-sdlc rebase <pr>` automates mechanical conflicts (CHANGELOG `Unreleased`, test additions to same `describe`, prettier drift) and re-signs the attestation only when `contentHash` changed. Escalates semantic conflicts, modify-vs-delete, verification failures, and 3-attempt iteration cap. Refuses force-push to `main`/`master`.

## CI marker hygiene

GitHub Actions silently skips ALL workflows when ANY commit body contains `[skip ci]`, `[ci skip]`, `[no ci]`, `[skip actions]`, or `[actions skip]` (substring, case-insensitive). Use the paren-quoted form `(skip ci marker)` in commit messages. Backtick-wrapping does NOT defeat the parser. `scripts/check-skip-ci-marker.sh` enforces on push.

## Branches & Commits

- Branches: `feat/<desc>`, `fix/<desc>`, or `ai-sdlc/issue-<n>`.
- Conventional commits (`feat:`, `fix:`, `test:`, `docs:`, `chore:`, `style:`).
- Include `Co-Authored-By: Claude Opus 4.6 (1M context) <noreply@anthropic.com>`.

## PRs

- **Never merge PRs** — only humans merge.
- **Never close** issues or PRs. **Never force-push to main/master.**
- Dismiss stale reviews only with documented reason (truncation, API errors).
- `auto-enable-auto-merge.yml` sets `--auto` on same-repo PRs (no method flag — the merge-queue ruleset on `main` enforces its configured strategy and overrides any flag passed by the workflow). Currently the queue is SQUASH so PRs land as one commit on main; if the queue's strategy is ever flipped in repo settings, no workflow change is needed. Setting `--auto` is NOT merging. (Legacy `--rebase` workaround retired in AISDLC-221 — GitHub now serializes auto-merge through the queue strategy, so the old "method-must-differ" trap no longer reproduces.)

## Testing

- Run `pnpm build && pnpm test && pnpm lint && pnpm format:check` before pushing.
- `.husky/pre-push` is the canonical gate; local pre-flight makes it a no-op.
- Hook scripts (`ai-sdlc-plugin/hooks/*.js`) use Node built-in `node --test`. Orchestrator + MCP server use Vitest.

## Hooks

`.husky/pre-push` chains in order:

1. **`scripts/check-coverage.sh`** — 80% lines coverage threshold per package. Skip: `AI_SDLC_SKIP_COVERAGE_GATE=1`.
2. **`scripts/check-task-moved.sh`** — auto-moves backlog task file from `backlog/tasks/` to `backlog/completed/` when any commit in the push range has `(AISDLC-N)` in its subject and the file is still in tasks/. Invokes the AISDLC-203 atomic helper (`cli-task-complete`), stages the move, commits as `chore: auto-close AISDLC-N (AISDLC-220)`, and exits 1 with "re-run git push". Idempotent on the second push (file already in completed/ or HEAD is auto-close chore predicate). **Order is load-bearing — MUST run BEFORE attestation-sign (item 3):** attestation's contentHashV4 binds `{path, headBlobSha}` per file; if the task move happens after sign, the envelope hashes the old path while the PR diff contains the new path → verify-attestation rejects. Skip: `AI_SDLC_SKIP_TASK_MOVE=1`.
3. **`scripts/check-attestation-sign.sh`** — auto-signs DSSE attestation when `<worktree>/.active-task` exists, `<worktree>/.ai-sdlc/verdicts/<task-id-lower>.json` exists, and no envelope at HEAD. Commits the envelope as a separate `chore: auto-sign attestation for <task-id>` and exits 1 with "re-run git push". Idempotent on the second push (envelope-at-HEAD or HEAD-is-auto-sign-chore predicate). **Docs-only auto-approve (AISDLC-215):** when the verdict file is missing AND `scripts/is-docs-only-changeset.mjs` reports the changeset is docs-only, the hook synthesizes a transient auto-approved verdicts file (3 reviewer entries, gitignored) and proceeds to sign — no manual step required. Code PRs with a missing verdict file still exit 0 (no-op) as before. Skip: `AI_SDLC_SKIP_ATTESTATION_SIGN=1`.

`set -euo pipefail` aborts on first failure. `git push --no-verify` bypasses everything. All gates have hermetic tests at `scripts/<name>.test.mjs` wired via `pnpm test:drift-gate` / `test:task-move-gate` / `test:attestation-sign-gate`.

## CI behavior

PR merge gate is the single rollup check `ai-sdlc/pr-ready` produced by `.github/workflows/ai-sdlc-gate.yml` (re-actors/alls-green pattern); see [`docs/operations/quality-gate.md`](docs/operations/quality-gate.md) for archetype gating, cutover, and rollback.

Workflows MUST invoke pipeline-cli CLIs via `node pipeline-cli/bin/cli-XXX.mjs` directly — never via `pnpm --filter @ai-sdlc/pipeline-cli exec cli-XXX`. `pnpm exec` does not resolve workspace own-bins, so the latter form silently fails with `Command not found` and any `|| echo <fallback>` safety net fires unconditionally. `pipeline-cli/src/cli/bin-invocation.test.ts` enforces both directions of this rule. See AISDLC-156 + the "Invoking from CI" section of `pipeline-cli/README.md`.

## Feature flags


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ai-sdlc-framework/ai-sdlc](https://github.com/ai-sdlc-framework/ai-sdlc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
