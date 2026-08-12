---
trigger: always_on
description: This file briefs every agent (codexu, claudeu, claude-code, opencode, or
---

# AGENTS.md — gormes-agent

This file briefs every agent (codexu, claudeu, claude-code, opencode, or
any future backend) that runs against this repository. Read it before
touching code or docs in `cmd/`, `internal/`, `webpages/docs/content/building-gormes/`,
or `progress.json`.

## Branch and CI safety rule

Main must always stay green. Treat this as a hard repository rule for every
agent in this workspace, including workspace-mineru / workspace-mimeru agents.

- `main` is protected release-trunk state. Do not do feature, docs, roadmap, or
  repair work directly on `main` after the bootstrap that created this rule.
- Work directly on the existing `development` branch only. Do not create or use
  short-lived branches, feature branches, or git worktrees for Gormes work.
  The release-prep GitHub Actions workflow may create `release/<version>` PR
  branches as an automation-only exception; agents must not create or use those
  branches for normal work.
- Before editing, confirm the current branch is `development`. If it is not,
  stop before changing files and switch safely to `development` or report the
  blocker; never create another branch or worktree as the workaround.
- Changes reach `main` only through a GitHub pull request into `main`.
- Before opening or updating a PR to `main`, run the same gate as CI:
  `go test ./... -count=1`, `go run ./cmd/progress validate`, and
  `git diff --check`.
- If `main` is red, stop normal feature work and repair through the
  `development` branch and PR path. Do not check out `main` for edits and do
  not branch new work from a red `main`.
- This branch rule overrides any generic agent or skill workflow that suggests
  temporary branches or git worktrees.
- GitHub rules for `main` must require pull requests and the required CI status
  check. Do not bypass them unless the user explicitly asks for emergency
  repository recovery.

## Mandatory repo-local skill routing

Before doing any substantive work in this repository, every agent must select
and use at least one repo-local skill. The canonical skill source is
`development-skills/<name>/SKILL.md`; `.agents/skills/`,
`.claude/skills/`, and `.codex/skills/` are symlink loader views for different
agents. If the right skill is not obvious, start with `gormes-skill-manager`
and let it route the task. Do not proceed "skill-less" on planning, building,
parity analysis, interface design, TDD implementation, or skill maintenance.

Use these skills as the default routing surface:

| Work type | Required skill |
|---|---|
| Unsure which workflow applies, or deciding whether a new skill is needed | `gormes-skill-manager` |
| Running a recurring full Hermes-in-Go parity sweep or recording periodic parity progress | `gormes-hermes-parity` as the subskill orchestrator |
| Discovering useful OpenClaw-only behavior absent from Hermes for possible Gormes-owned adoption | `gormes-openclaw-parity` |
| Discovering reusable Pi harness techniques without making Pi a parity contract | `gormes-pi-parity` |
| Mapping Hermes/Honcho parity gaps | `gormes-parity-auditor` |
| Fixing provider/auth/client/model-routing/usage/rate-limit parity bugs | `gormes-provider-parity` |
| Browser automation parity, Browser Use, browser-harness, CDP, or `/browser connect` work | `gormes-browser-harness` |
| Local run/install/runtime work: `go run ./cmd/gormes`, `bin/gormes`, `install.sh`, managed source checkouts, PATH shadowing, gateway process ownership, or `sessions.db` locks | `gormes-dev-runtime` |
| Updating roadmap rows, phases, dependencies, or planning docs | `gormes-planner` |
| Running a bounded architecture -> planner -> parity -> builder delivery cycle | `gormes-delivery-loop` |
| Implementing one `progress.json` row | `gormes-builder` |
| Red-green-refactor delivery of one behavior | `gormes-tdd-slice` |
| Designing Go package/API boundaries before implementation | `gormes-interface-designer` |
| Refactoring one `cmd/gormes` command domain into `internal/app/<domain>` without behavior changes | `cmd-internal-refactor` |
| Stuck on a Go implementation shape; want a donor file from `references/go-agent-os/` before writing code | `gormes-references` |
| External library/framework/upstream source context before planning or implementation | `gormes-context-sourcing` |
| Repeated runtime mechanics or service-layer cleanup after a feature works | `gormes-service-layer-refactor` |
| PR feedback, CI failures, or bounded review-to-green iteration | `gormes-review-loop` |
| Auditing or periodically refreshing README/public repository messaging | `gormes-readme` |
| Improving `www.gormes.ai` landing page content or UI | `gormes-landing-web` |
| Designing, critiquing, or polishing dashboard screenshots, hero images, social cards, or image-based dashboard assets | `dashboard-image-design` |
| Committing all dirty work, making `development` green, and pushing it | `gormes-git` |
| Preparing, PR-merging, tagging, and verifying a Gormes release | `gormes-release` |
| Stress-testing a plan or decision tree with the user | global `grill-me`; pair with repo-local `gormes-skill-manager` when Gormes routing is needed |

The global `/home/xel/.agents/skills/grill-me` skill is canonical; do not add a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TrebuchetDynamics/gormes-agent](https://github.com/TrebuchetDynamics/gormes-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
