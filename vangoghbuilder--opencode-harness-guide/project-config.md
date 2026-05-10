---
trigger: always_on
description: This repository is in an early documentation-first rewrite state.
---

# AGENTS.md

## Purpose

This repository is in an early documentation-first rewrite state.
This file gives coding agents a grounded operating guide while the public documentation identity is being rewritten toward `opencode-harness-guide`.
It separates verified facts from provisional conventions.

## Verified repository facts

- Repository path: `/Users/vangogh/Documents/code/opencode-howto`
- Public documentation identity in progress: `opencode-harness-guide`
- Current state: documentation-first rewrite in progress, with the content being reframed around harness engineering
- Present root docs: `AGENTS.md`, `README.md`, `CATALOG.md`, `STYLE_GUIDE.md`, `CONTRIBUTING.md`, `SECURITY.md`, `CODE_OF_CONDUCT.md`, `CHANGELOG.md`, `SUPPORT.md`, `.opencode/README.md`
- Present Chinese primary entry docs: `README.md`, `CATALOG.zh-CN.md`, `.opencode/README.md`, `examples/README.md`
- Present license-status file: `LICENSE`
- Present numbered module primary entry docs: `01-getting-started/README.md` through `10-cli-and-terminal/README.md`
- Present English auxiliary module entry docs: `01-getting-started/README.en.md` through `10-cli-and-terminal/README.en.md`
- Present starter templates:
  - `01-getting-started/templates/AGENTS.md`
  - `02-project-context/templates/PROJECT-FACTS-CHECKLIST.md`
  - `03-commands-and-prompts/templates/PLAN-REQUEST.md`
  - `03-commands-and-prompts/templates/REVIEW-REQUEST.md`
  - `03-commands-and-prompts/templates/COMMIT-REQUEST.md`
  - `03-commands-and-prompts/templates/PR-REQUEST.md`
  - `04-skills-and-agents/templates/SPECIALIZATION-DECISION-CHECKLIST.md`
  - `04-skills-and-agents/templates/skills/self-assessment/SKILL.md`
  - `05-hooks-and-automation/templates/AUTOMATION-BOUNDARY-CHECKLIST.md`
  - `06-integrations-and-mcp/templates/LOCAL-INTEGRATION-NOTES.md`
  - `07-team-workflows/templates/TEAM-ONBOARDING-CHECKLIST.md`
  - `08-cross-stack-templates/templates/STACK-STARTER-READINESS-CHECKLIST.md`
  - `09-advanced-workflows/templates/ADVANCED-WORKFLOW-CHECKLIST.md`
- No package manager is currently verified in this repository
- No package manifest was found
- No lint, test, typecheck, or build config was found
- No `.cursorrules` file was found
- No `.cursor/rules/**` files were found
- No `.github/copilot-instructions.md` file was found
- Present `.github` support files: `.github/pull_request_template.md`, `.github/SECURITY_REPORTING.md`, `.github/ISSUE_TEMPLATE/bug_report.md`, `.github/ISSUE_TEMPLATE/feature_request.md`, `.github/ISSUE_TEMPLATE/documentation.md`, `.github/ISSUE_TEMPLATE/question.md`, `.github/ISSUE_TEMPLATE/config.yml`

## Project direction

The intended direction is a rewrite of the project into `opencode-harness-guide`, still inspired by `/Users/vangogh/Documents/code/claude-howto` in structure, but centered on harness engineering rather than generic how-to onboarding.
That means documentation-first structure, a clear learning path, and copy-paste-ready templates.

### Target reader

The primary audience is people using OpenCode for the first time.
Assume they need a fast on-ramp, plain explanations, and examples that work as a starting point.

### Content scope

- Follow the spirit of `claude-howto`
- Ship **docs plus reusable configuration templates**, not docs alone
- Aim long-term for broad, cross-stack coverage

Treat that as direction, not as proof that equivalent files already exist here.

## Facts first, assumptions second

Before changing anything, sort statements into two buckets:

- **Verified fact**: supported by files that exist in this repository now
- **Provisional convention**: a temporary rule used until real project structure exists

When something is not yet established, label it as `TBD`, `Provisional`, or `Not yet present`.
Do not present intended future structure as current reality.

## Pre-flight checks

Before adding files, docs, or code, check these in order:

1. What files exist right now?
2. Is there already a README, roadmap, or module structure?
3. Is a stack choice documented anywhere?
4. Is a package manager or task runner configured?
5. Are lint, format, test, typecheck, or build commands defined anywhere?
6. Are Cursor or Copilot instruction files present?
7. Is the task asking for docs, scaffolding, templates, or executable code?

If the answer is no, say so plainly and take the smallest valid next step.

## Command status

### Verified current status

No install, dev, lint, test, single-test, typecheck, or build commands are currently verified in this repository.
There is no evidence yet of `npm`, `pnpm`, `yarn`, `bun`, `make`, `just`, or custom task scripts.

### Agent rules for commands

- Do not claim a command exists unless a real file defines it
- Do not invent `package.json`, `Makefile`, CI jobs, or script names as facts
- If a task needs tooling that does not exist yet, say that explicitly and add it only if the task asks for setup

### Placeholder wording

- `Install command: TBD once a package manager is selected.`
- `Lint command: TBD once lint tooling is added.`
- `Test command: TBD once a test framework is chosen.`
- `Single test command: TBD once the test runner is chosen.`
- `Build command: TBD once the project has buildable artifacts.`

## Editing rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VanGoghBuilder/opencode-harness-guide](https://github.com/VanGoghBuilder/opencode-harness-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
