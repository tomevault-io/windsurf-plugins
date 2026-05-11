---
trigger: always_on
description: ObsidiBot is an Obsidian plugin that runs a Claude Code CLI subprocess inside a vault,
---

# AGENTS.md — ObsidiBot Development Guide

ObsidiBot is an Obsidian plugin that runs a Claude Code CLI subprocess inside a vault,
providing an agentic chat panel for reading, writing, creating, moving, and organizing
notes. Desktop only (Windows, Mac, Linux). No mobile support.

-----

## Agent Identity

- GitHub username: `SKVFX-DevBot`
- Fork URL: `https://github.com/SKVFX-DevBot/ObsidiBot`
- Upstream URL: `https://github.com/ScottKirvan/ObsidiBot`
- Local clone: `~/ObsidiBot`

**Recommended invocation:**

```bash
claude --allowedTools "Read" "Edit" "Write" "Glob" "Grep" "Bash(git:*)" "Bash(npm:*)" "Bash(npx:*)" "Bash(gh:*)" "Bash(node:*)" "Bash(cd:*)" \
  -p "Clone https://github.com/SKVFX-DevBot/ObsidiBot to ~/ObsidiBot if it does not already exist, then read AGENTS.md from that directory and work on the next available issue."
```

**Verify `gh` authentication before doing anything else:**

```bash
gh auth status
```

If not authenticated, run `gh auth login` and select HTTPS when prompted. If auth fails and cannot be resolved, log the error and exit — do not proceed.

-----

## Repository Overview

GitHub: `https://github.com/ScottKirvan/ObsidiBot`

> The tree below may be outdated. The local clone is authoritative. Run `git ls-files` for the canonical file list.

```
ObsidiBot/
├── main.ts                      # Plugin entry point (root-level, compiled from src/)
├── main.js                      # Compiled output (gitignored in source control)
├── styles.css                   # Plugin stylesheet (gitignored in source control)
├── manifest.json                # Obsidian plugin manifest (do not edit version manually)
├── package.json
├── tsconfig.json
├── esbuild.config.mjs           # Build configuration
├── src/                         # TypeScript source
│   ├── ClaudeProcess.ts         # Claude Code CLI subprocess lifecycle
│   ├── ClaudeSession.ts         # Session state and serialization
│   ├── ClaudeView.ts            # Main chat panel (Obsidian ItemView)
│   ├── ContextGenerationModal.ts
│   ├── ContextManager.ts        # Vault context injection
│   ├── FrontmatterGuard.ts      # Prevents accidental frontmatter corruption
│   ├── QueryHandler.ts          # User input → subprocess routing
│   ├── UIBridge.ts              # @@CORTEX_ACTION dispatch layer
│   ├── constants.ts
│   ├── settings.ts              # Plugin settings tab and defaults
│   ├── declarations.d.ts        # Ambient type declarations
│   ├── modals/
│   │   ├── AboutModal.ts
│   │   ├── ExportToVaultModal.ts
│   │   └── SessionListModal.ts
│   └── utils/
│       ├── fileTree.ts          # Vault file tree generation for context
│       ├── logger.ts            # [ObsidiBot]-prefixed debug logging
│       ├── sessionStorage.ts    # Persist/load chat sessions
│       └── shellEnv.ts         # Shell environment resolution for subprocess
└── test/
    └── unit.test.ts             # Unit tests (Node built-in test runner via tsx)
```

-----

## Development Environment

**Prerequisites:**

- Node.js 18 or later (required for the built-in `--test` runner used by `npm test`)
- Claude Code CLI installed and authenticated
- Obsidian desktop with a test vault

**Setup:**

```bash
npm install
npm run dev       # watch mode, compiles main.ts → main.js
```

**Build:**

```bash
npm run build
```

**Tests:**

```bash
npm test          # runs: npx tsx --test test/unit.test.ts (Node built-in test runner)

# Run a single test by name pattern:
npx tsx --test --test-name-pattern="<pattern>" test/unit.test.ts
```

-----

## Session Pre-flight

Before selecting an issue or writing any code, run these checks in order. Stop and alert the user if any step fails.

```bash
# 1. Verify Node version (must be 18+)
node --version

# 2. Verify gh is authenticated as the agent account
gh auth status

# 3. Verify git remotes are correct
git remote -v   # origin → SKVFX-DevBot/ObsidiBot, upstream → ScottKirvan/ObsidiBot

# 4. Verify clean working tree on main
git status      # must show: On branch main, nothing to commit

# 5. Sync with upstream
git fetch upstream && git rebase upstream/main   # must succeed with no conflicts

# 6. Verify baseline tests pass before making any changes
npm test
```

If `gh auth status` fails, run `gh auth login` and select HTTPS. If any other step fails, stop and alert the user — do not proceed.

-----

## Fork Workflow

Contributions to ObsidiBot are made via personal forks. Do not push branches directly to
`ScottKirvan/ObsidiBot`. The upstream repository is `https://github.com/ScottKirvan/ObsidiBot`.

**Initial setup (once):**

```bash
# Fork ScottKirvan/ObsidiBot on GitHub, then:
git clone git@github.com:<your-account>/ObsidiBot.git
cd ObsidiBot
git remote add upstream https://github.com/ScottKirvan/ObsidiBot.git
```

**Before starting any issue — mandatory:**

```bash
git remote -v   # confirm 'origin' points to fork and 'upstream' points to ScottKirvan/ObsidiBot
git fetch upstream
git checkout main
git rebase upstream/main
git push origin main          # keep your fork's main current
```

If `upstream` is not set, run:

```bash
git remote add upstream https://github.com/ScottKirvan/ObsidiBot.git
```

Do not skip this step. PRs submitted against a stale base will be closed.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ScottKirvan/ObsidiBot](https://github.com/ScottKirvan/ObsidiBot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
