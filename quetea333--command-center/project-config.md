---
trigger: always_on
description: This repository builds apps where the AI agent and UI are equal partners:
---

# Agent-Native Framework

This repository builds apps where the AI agent and UI are equal partners:
everything the UI can do, the agent can do through the same SQL data and action
surface. Keep this file small: the rules below are the invariants, and
`.agents/skills/*` carries the workflow. Read the relevant skill before changing
that area.

## Skills

`.agents/skills/` holds the deep guidance, one directory per skill, each with a
`description` naming when to read it. Read the matching skill before changing
that area — most encode a decision the surrounding code cannot show. Prefer
searching the skill directory over guessing from nearby code. When a rule here
names a skill, that skill is the authority; this file only states the invariant.

A few are entry points rather than area guides:

- `adding-a-feature` — the four-area checklist every feature must satisfy.
- `content-product-development` — read before planning, implementing,
  reviewing, testing, or documenting Content behavior or shared framework
  behavior that changes Content's product contract.
- `writing-agent-instructions` — read before editing any `AGENTS.md`,
  `SKILL.md`, or tool/action description, including this file.
- `verifying-changes` — read before reporting a fix, feature, or deploy as
  done. Exercising the path that was broken is the step most often skipped,
  and skipping it is why the same bug gets reported twice.
- `reporting-progress` — read during any run over a few minutes, and at the
  moment you are tempted to stop and ask. Chasing status is the single most
  frequent correction in this repo.
- `concurrent-agents` — read before working in a shared checkout.
- `ship` — normal guarded ship through merge and branch rotation; beta and docs
  production deploys are automatic, while other production promotion is manual.
- `ship-and-monitor` — read when the normal ship flow also needs post-merge
  beta/release monitoring or explicit production-promotion verification.
- `ship-now` — fast admin-merge path with post-merge monitoring.

Spawning a read-only investigator? Use `/sidecar <task>` instead of retyping the
contract.

## Always-On Rules

- Scale effort to the task. A small, well-specified change is a short read, the
  edit, and the existing checks — not a codebase survey, unrequested tests, or
  browser automation. Save deep exploration for ambiguous or cross-cutting work.
- Stay on the current git branch. Never create, switch, delete, reset, rebase,
  stash, or otherwise move branches unless the user explicitly asks for that exact
  branch operation in the current task.
- Never add `Co-Authored-By` or other agent attribution to commits.
- PRs use the current branch unless the user explicitly requests a new branch.
  PRs are ready for review by default, not drafts, unless requested.
- Deployment split: `.github/workflows/deploy-beta-sites-prebuilt.yml` is the
  sole automatic beta publisher. It builds in GitHub Actions and uploads
  prebuilt artifacts to the independent Netlify beta sites at
  `beta.*.agent-native.com`; Netlify Git-connected auto-builds are disabled.
  Do not wait for Netlify build queues or deploy-preview checks. Verify the
  GitHub Actions run and its per-site smoke checks instead. Normal `/ship` does
  not monitor post-merge deployments or claim beta health; use
  `/ship-and-monitor` to verify beta. The public docs site is the temporary
  production exception: `.github/workflows/deploy-docs-production.yml` builds
  and publishes `fw` / `www.agent-native.com` from matching `main` changes,
  then disables the site's Git-connected Netlify builds. There is no beta docs
  site or beta docs hostname today. Other production promotion is manual, and
  critical fixes must be explicitly promoted to production through the manual
  `.github/workflows/deploy-production-sites-prebuilt.yml` or targeted
  `promote-netlify-deploy.yml` workflows. Let the workflow manage Netlify lock
  transitions; do not manually remove a lock or imply that clearing one makes
  production live.
- Worktrees are valid PR sources. When the user authorizes shipping or opening
  or updating a PR from a worktree, use that worktree's current branch and cwd
  for the commit, push, and PR operation; do not copy changes into the shared
  checkout.
- Use root `.tmp/` for repo-local temp files; it is gitignored.
- Never use `[codex]`, `codex`, or similar agent labels in user-visible GitHub
  metadata unless explicitly requested.
- Keep the chat title accurate.
- Do the work instead of asking whether to do it. If a step is inside the task
  you were given and is not destructive, irreversible, or a spend/send/publish
  action, run it and report the result — deploys, database reads and writes,
  scripts, and browser checks included. Ask only for a missing credential, a
  decision only the user can make, or a destructive action. `reporting-progress`
  is the authority on what to post while long work runs and on the one legal
  shape of a mid-task stop.
- Use sub-agents liberally for complex independent work when Agent Teams are
  available; keep the main thread focused on orchestration.
- When adding package dependencies or framework integrations, verify the current

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QueTea333/Command-Center](https://github.com/QueTea333/Command-Center) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
