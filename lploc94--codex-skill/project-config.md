---
trigger: always_on
description: This repository provides a single-command installer (`npx github:lploc94/codex_skill`) that installs the `codex-review` skill pack into `~/.claude/skills/codex-review/`.
---

# CLAUDE.md

This repository provides a single-command installer (`npx github:lploc94/codex_skill`) that installs the `codex-review` skill pack into `~/.claude/skills/codex-review/`.

## Project Overview

`codex-review` provides eight skills powered by OpenAI Codex CLI:
- `/codex-plan-review` — debate plans before implementation
- `/codex-impl-review` — review uncommitted or branch changes before commit/merge
- `/codex-think-about` — peer reasoning/debate on technical topics
- `/codex-commit-review` — review committed code quality before push (report + suggest)
- `/codex-pr-review` — review PRs (branch diff, commit hygiene, description)
- `/codex-parallel-review` — parallel independent review by both Claude and Codex, then debate
- `/codex-codebase-review` — chunked full-codebase review for large projects (50-500+ files)
- `/codex-security-review` — security-focused review using OWASP Top 10 and CWE patterns



## Distribution Model

- Single command install: `npx github:lploc94/codex_skill`
- Installs to: `~/.claude/skills/codex-review/`
- No global npm install, no CLI left behind, no node_modules on user machine

## Requirements

- Node.js >= 22
- Claude Code CLI
- OpenAI Codex CLI in PATH (`codex`)
- OpenAI API key configured for Codex

## Development Commands

```bash
node ./bin/codex-skill.js                                          # run installer locally
node ./bin/codex-skill.js --auto                                   # install + inject guidance into ~/.claude/CLAUDE.md
node skill-packs/codex-review/scripts/codex-runner.js version      # runner version
```

There is no build system, test suite, or linter. The project is JavaScript + Markdown + JSON.

## Architecture

### Installer

`bin/codex-skill.js` — single file, Node.js stdlib only, no dependencies:
1. Parse CLI arguments (`-full`, `--auto`)
2. Runtime guard: Node.js >= 22
3. Build staging directory alongside install target
4. Copy `codex-runner.js` from `skill-packs/`
5. Read SKILL.md templates (contain `{{RUNNER_PATH}}`), inject absolute path, write to staging
6. Copy `references/` directories as-is
7. Verify runner by spawning `node codex-runner.js version`
8. Atomic swap: backup old install → rename staging → cleanup
9. If `--auto`: inject review guidance into `~/.claude/CLAUDE.md` (idempotent, append-only)

### Skill Pack Layout (templates + runner)

```text
skill-packs/codex-review/
├── manifest.json
├── scripts/
│   └── codex-runner.js          ← single shared Node.js runner
└── skills/
    ├── codex-plan-review/
    │   ├── SKILL.md             ← template with {{RUNNER_PATH}}
    │   └── references/
    ├── codex-impl-review/
    │   ├── SKILL.md
    │   └── references/
    ├── codex-think-about/
    │   ├── SKILL.md
    │   └── references/
    ├── codex-commit-review/
    │   ├── SKILL.md
    │   └── references/
    ├── codex-pr-review/
    │   ├── SKILL.md
    │   └── references/
    ├── codex-parallel-review/
    │   ├── SKILL.md
    │   └── references/
    ├── codex-codebase-review/
    │   ├── SKILL.md
    │   └── references/
    └── codex-security-review/
        ├── SKILL.md
        └── references/
```

### Installed Output (on user machine)

```text
~/.claude/skills/
├── codex-review/
│   └── scripts/
│       └── codex-runner.js              ← shared runner
├── codex-plan-review/
│   ├── SKILL.md                         ← RUNNER="/abs/path/codex-runner.js" hardcoded
│   └── references/
├── codex-impl-review/
│   ├── SKILL.md
│   └── references/
├── codex-think-about/
│   ├── SKILL.md
│   └── references/
├── codex-commit-review/
│   ├── SKILL.md
│   └── references/
├── codex-pr-review/
│   ├── SKILL.md
│   └── references/
├── codex-parallel-review/
│   ├── SKILL.md
│   └── references/
├── codex-codebase-review/
│   ├── SKILL.md
│   └── references/
└── codex-security-review/
    ├── SKILL.md
    └── references/
```

### Core Execution Flow

1. **Skill invocation** (`/codex-plan-review`, `/codex-impl-review`, `/codex-think-about`, `/codex-commit-review`, `/codex-pr-review`, `/codex-parallel-review`, `/codex-codebase-review`, or `/codex-security-review`) follows SKILL.md step-by-step
2. **Runner path**: SKILL.md contains hardcoded absolute path to `codex-runner.js`
3. **Prompt rendering**: SKILL.md calls `render --skill X --template Y --skills-dir $SKILLS_DIR` with JSON vars on stdin → receives rendered prompt on stdout
4. **Session lifecycle**: `init` → `start` (stdin prompt) → `poll` (JSON response) → `resume` (stdin prompt) → `poll` → ... → `finalize` → `stop`
5. **codex-runner.js** spawns `codex exec --json --sandbox read-only` as a detached process, polls JSONL output, parses markdown into structured JSON
6. **Review debate loop** (plan-review, impl-review, commit-review, pr-review): Claude reads `poll` JSON → `review.blocks[].id` ISSUE-{N} → fixes/rebuts → `render` rebuttal → `resume` → repeats until `APPROVE` or stalemate
7. **Peer debate loop** (think-about): Claude and Codex think independently → compare JSON responses → exchange perspectives → repeat until consensus or stalemate → present to user

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lploc94/codex_skill](https://github.com/lploc94/codex_skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
