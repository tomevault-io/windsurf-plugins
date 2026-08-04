---
trigger: always_on
description: **Purpose:** Single source of truth for any AI coding assistant (Claude Code,
---

# AGENTS.md — Documentation for AI Agents working on this repo

**Purpose:** Single source of truth for any AI coding assistant (Claude Code,
Cursor AI, OpenAI Codex, Gemini CLI, GitHub Copilot, others) that **edits
this repository**. This file is for contributors, not for end users of the
skill.

> [!IMPORTANT]
> **This file is NOT installed.** When users install the skill (via
> `npx skills add`, `openclaw skills install`, or `git clone + setup.sh`),
> only the contents of `skills/deepworkplan/` ship to their machine. Anything
> outside that directory — including this file, `.github/`, `tests/`,
> `scripts/`, `docs/`, `README.md`, `LICENSE`, etc. — is repo-development
> infrastructure that lives only on GitHub and on contributors' machines.

`CLAUDE.md` at the repo root is a symlink to this file so Claude Code reads
the same instructions other agents do.

---

## Detailed Documentation

| Category | Document |
|----------|----------|
| User-facing README | [README.md](README.md) |
| Human contributor guide (narrative companion to this file) | [CONTRIBUTING.md](CONTRIBUTING.md) |
| Design decisions (the *why* behind the layout) | [docs/DESIGN.md](docs/DESIGN.md) |
| Install guide (compare / update / uninstall) | [docs/INSTALLATION.md](docs/INSTALLATION.md) |
| OpenClaw-specific notes | [docs/OPENCLAW.md](docs/OPENCLAW.md) |
| Adding a new sub-skill (step-by-step) | [docs/SUB_SKILL_GUIDE.md](docs/SUB_SKILL_GUIDE.md) |
| Security posture (secrets handling, boundaries, dogfooded review) | [docs/SECURITY.md](docs/SECURITY.md) |
| Security policy (report a vulnerability) | [SECURITY.md](SECURITY.md) |
| Changelog | [CHANGELOG.md](CHANGELOG.md) |
| Launch / publishing playbook | [PUBLISHING.md](PUBLISHING.md) |
| Router meta-skill (version source of truth) | [skills/deepworkplan/SKILL.md](skills/deepworkplan/SKILL.md) |
| Normative specification (the standard; 5 RFC-2119 docs) | [skills/deepworkplan/spec/](skills/deepworkplan/spec/README.md) |
| `create` sub-skill | [skills/deepworkplan/create/SKILL.md](skills/deepworkplan/create/SKILL.md) |
| `execute` sub-skill | [skills/deepworkplan/execute/SKILL.md](skills/deepworkplan/execute/SKILL.md) |
| `refine` sub-skill | [skills/deepworkplan/refine/SKILL.md](skills/deepworkplan/refine/SKILL.md) |
| `resume` sub-skill | [skills/deepworkplan/resume/SKILL.md](skills/deepworkplan/resume/SKILL.md) |
| `status` sub-skill | [skills/deepworkplan/status/SKILL.md](skills/deepworkplan/status/SKILL.md) |
| `verify` sub-skill (conformance check) | [skills/deepworkplan/verify/SKILL.md](skills/deepworkplan/verify/SKILL.md) |
| `onboard` sub-skill (make any repo AI-first) | [skills/deepworkplan/onboard/SKILL.md](skills/deepworkplan/onboard/SKILL.md) |
| `author` sub-skill (author/update skills, agents, commands) | [skills/deepworkplan/author/SKILL.md](skills/deepworkplan/author/SKILL.md) |
| Methodology guide | [skills/deepworkplan/guide/GUIDE.md](skills/deepworkplan/guide/GUIDE.md) |
| Context detection + `.dwp/` resolution | [skills/deepworkplan/shared/context.sh](skills/deepworkplan/shared/context.sh) |
| `.dwp/` output path convention | [skills/deepworkplan/shared/dwp-paths.md](skills/deepworkplan/shared/dwp-paths.md) |
| Reasoning-over-copy-paste principle | [skills/deepworkplan/shared/adaptation.md](skills/deepworkplan/shared/adaptation.md) |
| Onboarding presets (per-stack) | [skills/deepworkplan/onboard/presets/](skills/deepworkplan/onboard/presets/README.md) |
| Devcontainer addon (opt-in) | [skills/deepworkplan/addons/devcontainer/SKILL.md](skills/deepworkplan/addons/devcontainer/SKILL.md) |
| Dailybot addon (opt-in) | [skills/deepworkplan/addons/dailybot/SKILL.md](skills/deepworkplan/addons/dailybot/SKILL.md) |
| AI Diff Reviewer addon (opt-in) | [skills/deepworkplan/addons/ai-diff-reviewer/SKILL.md](skills/deepworkplan/addons/ai-diff-reviewer/SKILL.md) |
| Dependency Upgrade addon (opt-in) | [skills/deepworkplan/addons/dependency-upgrade/SKILL.md](skills/deepworkplan/addons/dependency-upgrade/SKILL.md) |
| Design System addon (opt-in) | [skills/deepworkplan/addons/design-system/SKILL.md](skills/deepworkplan/addons/design-system/SKILL.md) |
| Workflows reference (`auto-release`, `ci`, `pr-review`) | [.github/docs/WORKFLOWS.md](.github/docs/WORKFLOWS.md) |

## Project Overview

This repository is the **official DeepWorkPlan agent skill pack**, maintained
by [Dailybot](https://www.dailybot.com) and distributed via
[skills.sh](https://skills.sh), [OpenClaw](https://www.openclaw.dev), and
direct git clone. It is a **Markdown-first** skill: it teaches AI coding agents
how to (1) turn any repository AI-first — generating an adapted `AGENTS.md`,
`docs/`, per-module docs, `.agents/`, and the `.claude → .agents` /
`.cursor → .agents` symlinks — and
(2) run structured multi-task **Deep Work Plans** whose outputs land in a
gitignored `.dwp/` directory. The skill follows the
[Open Agent Skills](https://agentskills.io) standard.

**Stack:** Markdown + Bash. No application runtime, no compiled artifacts.
The "code" is the `SKILL.md` prompt files an agent reads at runtime, plus two
small helper scripts: `setup.sh` (the symlink installer at the repo root) and
`skills/deepworkplan/shared/context.sh` (repo/branch/agent + `.dwp/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DailybotHQ/deepworkplan-skill](https://github.com/DailybotHQ/deepworkplan-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
