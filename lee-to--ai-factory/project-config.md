---
trigger: always_on
description: > This file is for AI agents working on this codebase. Read this first when starting a new session.
---

# AI Factory - Developer Guide

> This file is for AI agents working on this codebase. Read this first when starting a new session.

## What is this project?

**AI Factory** (v2) is an npm package + skill system that automates AI agent context setup for projects. It provides:

1. **CLI tool** (`ai-factory init/update/upgrade`) — installs skills and configures MCP
2. **Built-in skills** (24 skills, all `aif-*` prefixed) — workflow commands for spec-driven development
3. **Spec-driven workflow** — structured approach: plan → implement → commit
4. **Multi-agent support** — 15 agents (Claude Code, Cursor, Windsurf, Roo Code, Kilo Code, Antigravity, OpenCode, Warp,
   Zencoder, Codex CLI, GitHub Copilot, Gemini CLI, Junie, Qwen Code, Universal)

## Project Structure

```
ai-factory/
├── src/                    # CLI source (TypeScript)
│   ├── cli/
│   │   ├── commands/       # init.ts, update.ts, upgrade.ts
│   │   └── wizard/         # prompts.ts
│   ├── core/               # installer.ts, config.ts, mcp.ts, agents.ts, template.ts, transformer.ts
│   │   └── transformers/   # default.ts, antigravity.ts, kilocode.ts
│   └── utils/              # fs.ts
├── skills/                 # Built-in skills (copied to user projects)
│   ├── aif/                           # Main setup skill
│   ├── aif-architecture/       # Architecture patterns
│   ├── aif-best-practices/     # Code quality guidelines
│   ├── aif-build-automation/   # Makefile/Taskfile/Justfile generator
│   ├── aif-ci/                 # GitHub Actions / GitLab CI generator
│   ├── aif-commit/             # Conventional commits
│   ├── aif-dockerize/          # Docker/compose generator
│   ├── aif-docs/               # Documentation generation & maintenance
│   ├── aif-evolve/             # Self-improve skills based on context
│   ├── aif-explore/            # Explore mode (thinking partner)
│   ├── aif-fix/                # Quick bug fixes (no plans)
│   ├── aif-grounded/           # Reliability gate for answers
│   ├── aif-implement/          # Execute plan tasks
│   ├── aif-improve/            # Plan refinement (second iteration)
│   ├── aif-loop/               # Iterative reflex loop with quality gates
│   ├── aif-plan/               # Plan implementation (fast/full modes)
│   ├── aif-review/             # Code review
│   ├── aif-roadmap/            # Strategic project roadmap
│   ├── aif-rules/              # Project rules and conventions
│   ├── aif-qa/                 # QA workflow: change summary → test plan → test cases
│   ├── aif-security-checklist/ # Security audit
│   ├── aif-skill-generator/    # Generate new skills
│   └── aif-verify/             # Verify implementation against plan
├── scripts/                # test-skills.sh
├── mcp/                    # MCP server templates
├── dist/                   # Compiled JS
└── bin/                    # CLI entry point
```

## Key Concepts

### Skills Location

- **Package skills**: `skills/` — source of truth, copied during install
- **User skills**: `<agent-config-dir>/skills/` (e.g. `.claude/skills/`, `.opencode/skills/`, `.agents/skills/`)
- **Agent transformer system**: `src/core/transformers/` adapts skill format per agent (e.g. Antigravity uses flat `.md`
  for workflow skills, KiloCode sanitizes dotted names)

### Working Directory

All AI Factory files in user projects go to `.ai-factory/` by default. `config.yaml` can relocate many workflow
artifacts, but the paths below remain the default layout:

- `.ai-factory/DESCRIPTION.md` — project specification
- `.ai-factory/ARCHITECTURE.md` — architecture decisions and guidelines
- `.ai-factory/PLAN.md` — task plan (from /aif-plan fast)
- `.ai-factory/plans/<branch-or-slug>.md` — plans (from /aif-plan full)
- `.ai-factory/skill-context/<skill>/SKILL.md` — project-specific overrides for skills (from /aif-evolve)
- `.ai-factory/evolutions/*.md` — evolution logs (from /aif-evolve)
- `.ai-factory/evolutions/patch-cursor.json` — incremental evolve cursor (latest processed patch)
- `.ai-factory/qa/<branch-slug>/change-summary.md` — QA change summary (from /aif-qa)
- `.ai-factory/qa/<branch-slug>/test-plan.md` — QA test plan (from /aif-qa)
- `.ai-factory/qa/<branch-slug>/test-cases.md` — QA test cases (from /aif-qa)
- `.ai-factory/evolution/current.json` — active loop pointer (from /aif-loop)
- `.ai-factory/evolution/<alias>/run.json` — current loop state
- `.ai-factory/evolution/<alias>/history.jsonl` — loop event history (append-only)
- `.ai-factory/evolution/<alias>/artifact.md` — latest loop artifact output

### Artifact Ownership Contract

Artifact writers are command-scoped to prevent ownership conflicts:

| Artifact                                                                                      | Primary writer command | Notes                                                                                            |
|-----------------------------------------------------------------------------------------------|------------------------|--------------------------------------------------------------------------------------------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lee-to/ai-factory](https://github.com/lee-to/ai-factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
