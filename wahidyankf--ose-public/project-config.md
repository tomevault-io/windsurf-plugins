---
trigger: always_on
description: > ⚠️ **IMPORTANT**: This file documents `.opencode/` configuration which is **AUTO-GENERATED** from `.claude/` (source of truth).
---

# AGENTS.md

> ⚠️ **IMPORTANT**: This file documents `.opencode/` configuration which is **AUTO-GENERATED** from `.claude/` (source of truth).
>
> **To make changes**:
>
> 1. Edit agents/skills in `.claude/` directory
> 2. Run: `npm run sync:claude-to-opencode`
> 3. Changes will be synced to `.opencode/` automatically
>
> **See [CLAUDE.md](./CLAUDE.md) for primary documentation** (Claude Code configuration).

**Problem**: Maintaining quality and consistency across many specialized agents, skills, and extensive documentation is time-consuming and error-prone when done manually.

**Solution**: This repository uses specialized AI (Artificial Intelligence) agents that automate documentation creation, validation, content generation, and project planning—ensuring consistent quality, catching errors early, and freeing developers to focus on high-value work.

---

Instructions for AI agents working with this repository via OpenCode.

## Project Overview

**open-sharia-enterprise** - Enterprise platform built with Node.js, using **Nx monorepo** structure.

- **Node.js**: 24.13.1 (LTS - Long-Term Support, managed by Volta)
- **npm**: 11.10.1
- **Monorepo**: Nx with `apps/` and `libs/` structure
- **Git Workflow**: Trunk Based Development (default: commit and push directly to `main`). Running inside a git worktree does **not** change this default — the same direct-push-to-main rule applies whether the work executes in a worktree session or the main checkout. A draft PR is opt-in only: use one when the user's prompt explicitly requests a PR, or when the delivery checklist contains an explicit PR step that the user has confirmed. See the [Trunk Based Development Convention](./governance/development/workflow/trunk-based-development.md#default-push-and-worktree-execution) for the decision table and full details.
- **Worktree toolchain init**: After creating or entering a worktree, agents must run BOTH `npm install` AND `npm run doctor -- --fix` in the root repository worktree, in that order. The `package.json` `postinstall` hook runs `npm run doctor || true` which silently tolerates toolchain drift, so the explicit `doctor --fix` invocation is required to converge the 18+ polyglot toolchains (Go, Java, Rust, Elixir, Python, .NET, Dart, Clojure, Kotlin, C#, Node). See [Worktree Toolchain Initialization](./governance/development/workflow/worktree-setup.md) for the full rationale and procedure.

## Dual-Mode Configuration

This repository maintains **dual compatibility** with both Claude Code and OpenCode:

- **`.claude/`**: Source of truth (PRIMARY) - Edit here first
- **`.opencode/`**: Auto-generated (SECONDARY) - Synced from `.claude/`

**Sync Command**: `npm run sync:claude-to-opencode`

**Format Differences**:

- **Tools**: Claude Code uses arrays `[Read, Write]`, OpenCode uses `{ read: true, write: true }`
- **Models**: Claude Code uses `sonnet`/`haiku` or omits `model:` entirely (omit = budget-adaptive opus-inherit); OpenCode uses `zai-coding-plan/glm-5.1` or `zai-coding-plan/glm-5-turbo`
- **Skills**: Same format for both systems (SKILL.md)
- **Permissions**: Claude Code uses `settings.json`, OpenCode uses `opencode.json` permission block (equivalent access configured)
- **MCP/Plugins**: Claude Code uses plugins, OpenCode uses MCP servers (Playwright, Nx, Z.ai, Perplexity)

# AI Agents

## Agent Organization

Specialized agents organized into families:

1. **Documentation**: `docs-maker`, `docs-checker`, `docs-fixer`, `docs-tutorial-maker`, `docs-tutorial-checker`, `docs-tutorial-fixer`, `docs-link-checker`, `docs-file-manager`, `docs-software-engineering-separation-checker`, `docs-software-engineering-separation-fixer`
2. **README**: `readme-maker`, `readme-checker`, `readme-fixer`
3. **Project Planning**: `plan-maker`, `plan-checker`, `plan-execution-checker`, `plan-fixer` (plan execution itself is orchestrated directly by the calling context via the [plan-execution workflow](./governance/workflows/plan/plan-execution.md); no dedicated executor subagent)
4. **AyoKoding Web Content**: Bilingual content creators, validators, deployers (includes in-the-field agents: `apps-ayokoding-web-in-the-field-maker`, `apps-ayokoding-web-in-the-field-checker`, `apps-ayokoding-web-in-the-field-fixer`)
5. **Web Content - oseplatform-web**: Landing page content creators, validators, deployers (migrated from Hugo to Next.js 16)
6. **Software Engineering & Specialized**: `agent-maker`, `swe-code-checker`, `swe-ui-maker`, `swe-ui-checker`, `swe-ui-fixer`, `swe-clojure-dev`, `swe-csharp-dev`, `swe-dart-dev`, `swe-e2e-dev`, `swe-elixir-dev`, `swe-fsharp-dev`, `swe-golang-dev`, `swe-hugo-dev` (DEPRECATED), `swe-java-dev`, `swe-kotlin-dev`, `swe-python-dev`, `swe-rust-dev`, `swe-typescript-dev`, `social-linkedin-post-maker`, `apps-organiclever-web-deployer`, `apps-wahidyankf-web-deployer`
7. **Repository Governance**: `repo-rules-maker`, `repo-rules-checker`, `repo-rules-fixer`, `repo-workflow-maker`, `repo-workflow-checker`, `repo-workflow-fixer`, `repo-ose-primer-adoption-maker`, `repo-ose-primer-propagation-maker`
8. **Specs Validation**: `specs-maker`, `specs-checker`, `specs-fixer`
9. **CI/CD**: `ci-checker`, `ci-fixer`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wahidyankf/ose-public](https://github.com/wahidyankf/ose-public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
