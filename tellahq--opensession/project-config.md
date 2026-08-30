---
trigger: always_on
description: Default to Bun instead of Node.js.
---

Default to Bun instead of Node.js.

Keep instance-private operator instructions in an untracked `AGENTS.local.md` or
`CLAUDE.local.md`, never in this file.

## Publishing to repositories

Repositories owned by your own organization are fair game, including public
ones: commit, push, and open issues, comments, and pull requests there as part
of normal work.

For any repository outside your organization, public or third-party, never
publish without explicit user confirmation in the current conversation. Local
investigation and commits are allowed; issues, comments, branches, forks,
pushes, and pull requests are not.

## Choose the client first

Open Session has five clients:

- Web UI: `packages/core/opensession-server/src/frontend/`
- Phone web/PWA: the same web bundle at phone width
- Electron shell: `packages/clients/mac/`
- Native Swift app: `packages/clients/ios/`
- Chrome extension: `packages/clients/chrome/`

Once a conversation names a client, keep working on that client unless the user
changes scope. Ask when the target is unclear. For web changes, build desktop
and phone together. For protocol, preference, or transcript changes, check the
native app and Chrome extension for matching wire models or behavior. Read the
nearest nested `AGENTS.md` before editing a client.

## Shared checkout and deployment workflow

Sessions edit the shared `main` checkout, but the live services run from an
immutable release worktree selected by `~/.opensession/deploy/current`. Other
sessions may edit and stage files in the shared checkout at the same time.
Uncommitted checkout edits never become live, including frontend edits.

- Start every task by pulling the latest remote history with
  `git fetch origin --prune` and checking
  `git rev-list --left-right --count HEAD...origin/main`. Do not start or
  continue edits from a stale or diverged `main`.
- **Do not commit until your branch includes the latest `origin/main`.**
  Immediately before every `git commit`, run `git fetch origin --prune`. Then
  run `git merge-base --is-ancestor origin/main HEAD`. If it fails, do not
  commit: rebase the local commits onto `origin/main` and resolve every conflict
  first. Fetch again immediately before pushing; if the remote moved after your
  commit, rebase that commit onto the new `origin/main` before pushing.
- Keep one session responsible for synchronizing the shared checkout at a time.
  Preserve every staged, unstaged, and untracked change while rebasing. Never
  use `git stash`, autostash, `git pull --rebase --autostash`, reset, clean,
  force-push, or an ordinary pull that creates a merge commit in this checkout.
  After pushing, fetch once more and verify
  `git rev-list --left-right --count HEAD...origin/main` reports `0 0` before
  continuing.
- Never reset, revert, switch branches, or discard unrelated work.
- Stage only your files. Use `git add -p` for shared high-traffic files.
- Inspect `git diff --cached --name-only` and `git diff --cached` before every
  commit. Commit with a pathspec when the index contains other work.
- Commit and push promptly. Never use `git add -A`.
- Do not use an ad-hoc `systemctl restart`. It only restarts the already pinned
  release and can violate the gateway/kernel rollout order.
- Commit and push before deploying. Deployment may be autonomous when the task
  calls for making the change live, but it is a shared operation, not a
  per-session completion ritual. Concurrent main-line requests queue and
  coalesce to the newest fast-forward commit; targets already covered become
  successful no-ops. Do not manually retry a queued request.
- Treat a successful deploy as fully settled. Success already means the target
  release is active and its required health checks passed; the rollback safety
  window is protection, not a cooldown. Continue immediately with the next
  promotion or verification. Do not wait for a shared deploy to "settle", add
  an arbitrary delay, or wait out the safety window. If a lifecycle operation
  still owns the deploy lock, submit the normal deploy request and let it queue
  and coalesce instead of polling or sleeping.
- Use `deploy_self` for ordinary frontend, backend, protocol, and dependency
  changes. It classifies the complete diff from the running backend pin. A
  strictly frontend-only diff is built in its immutable target release and
  promoted without restarting any service; everything else uses the standard
  health-gated executor, session-kernel, and gateway restart. The deploy
  controller collapses overlapping main-line requests instead of producing a
  restart train.
- Use the full root deploy, `sudo deploy/deploy.sh <sha>`, instead when a change
  affects live deployment machinery or an artifact that script installs:
  `deploy/{deploy,self-deploy,release-checkout}.sh`, the four
  `opensession*.service` templates, credential installers, the fixed run-host
  helper/installer, or root-deploy-managed systemd units and drop-ins. The full
  deploy refreshes those privileged artifacts before switching the same
  immutable release. Other operator-managed artifacts, such as watchdog units
  and sandbox images, follow their own documented rollout. When unsure, inspect
  `deploy/deploy.sh` rather than assuming a restart applies the change.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tellahq/opensession](https://github.com/tellahq/opensession) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
