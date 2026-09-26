---
trigger: always_on
description: This repo contains patterns for bootstrapping AI-friendly projects. Follow these instructions to scaffold a complete new project for the developer.
---

# Agent Project Bootstrap

This repo contains patterns for bootstrapping AI-friendly projects. Follow these instructions to scaffold a complete new project for the developer.

> Scaffolding a **new** project. For applying these patterns to an **existing** codebase, see `ADOPT.md`.

## Step 0: Detect what's already installed

Hooks and skills install **system-wide** under `~/.claude/`, so they're shared
across every project. Detect them first and never ask about components that are
already present. Run:

```bash
# Hooks: install.sh stamps this file with the installed version
HOOKS_VER=$( [ -f ~/.claude/hooks/.agent-starter-version ] && cat ~/.claude/hooks/.agent-starter-version || echo "" )
HOOKS_N=$( ls ~/.claude/hooks/*.sh 2>/dev/null | wc -l | tr -d ' ' )

# Skills: the starter skills this bootstrap installs
for s in commit commit-push-pr simplify remember dream new-project adopt-project reflect; do
  [ -d ~/.claude/skills/$s ] && echo "skill:$s present" || echo "skill:$s missing"
done
echo "hooks: version ${HOOKS_VER:-none}, $HOOKS_N scripts"
```

Hooks are installed if `.agent-starter-version` exists (record the version);
skills are installed per directory listed as `present`. Carry this into the
interview and scaffold: only ask about, and only install, what's **missing**. If
a stamped hooks version is present but older than the repo `VERSION`, note an
update is available and offer to re-run `install.sh` (idempotent) - don't force it.

## Step 1: Interview the Developer

Ask these questions **one at a time** before taking any action:

1. **Project name** - what is the name of the project?
2. **Description** - one sentence describing what it does.
3. **Tech stack** - language, framework, package manager (e.g. "TypeScript, Next.js, pnpm").
   - **Tailwind design-system lint** - ask this follow-up only when the stack is
     TypeScript/JavaScript with a UI framework (React, Next.js, Remix, Vite +
     React): "Does the project use Tailwind v4, and should I add `@shadcn/lint`
     (blocks raw palette colors, arbitrary values, inline styles, unknown
     classes, and restyling design-system components via `className`)? yes/no".
     Record the answer; it drives the optional block in Step 2 step 4. Skip
     the question for non-UI stacks.
4. **Optional components** - ask **only about what Step 0 reported as missing**.
   If hooks and all skills are already installed, skip this question entirely -
   state what was detected and move on. Otherwise offer the missing set:
   - Hooks (auto-enforce file size limits, lint-on-save, silent-error and dangerous-command blocking, codebase health checks at `~/.claude/hooks/`)
   - Skills (commit, commit-push-pr, simplify, remember, dream, new-project, adopt-project, reflect at `~/.claude/skills/`)
   - Both
   - Neither
5. **Repo path** - what is the local path to the agent-starter repo? (e.g. `~/code/agent-starter`). Always required: the CLAUDE.md template, foundation templates, and lint configs are all copied from the repo. (Hooks and skills also install from here when selected and not already present.)

Do not proceed past this step until you have all answers.

## Step 2: Scaffold the Project

Execute these steps in order. Read the referenced files in this repo for full detail on each pattern.

### 1. Create directory structure

Reference: `guides/large-codebase-best-practices.md` - Section 1 (Feature-based directory structure)

Create the project root and subdirectories:

```
<project-name>/
├── src/
│   ├── features/      # feature modules - each gets its own directory
│   ├── services/      # shared business logic by domain
│   ├── utils/         # truly shared utilities
│   ├── types/         # shared type definitions (break import cycles here)
│   ├── constants/     # named constants by domain
│   ├── schemas/       # validation schemas
│   ├── entrypoints/   # app entry points
│   └── migrations/    # data/config format migrations
├── tests/
├── docs/
└── scripts/
```

### 2. Generate CLAUDE.md

Reference: `templates/CLAUDE.md`

Copy `templates/CLAUDE.md` into `<project-name>/CLAUDE.md`.
In the `## Project-Specific Instructions` section at the bottom, add:

```
**Project:** <project-name>
**Description:** <project-description>
```

### 3. Create config files

**`.gitignore`:**
```
node_modules/
dist/
.env
*.log
.DS_Store
.cache/
coverage/
CLAUDE.local.md
```

**`.env.example`:**
```
# Required environment variables - copy to .env and fill in values
```

**`README.md`:**
```markdown
# <project-name>

<project-description>

## Getting Started

<!-- Add setup instructions here -->
```

**`CLAUDE.local.md`** (gitignored - personal, machine-local instructions that are never committed): create it with just a comment header.

**`.claude/rules/`** - modular instruction files loaded alongside CLAUDE.md. Create `.claude/rules/starter-patterns.md`, the apply-on-touch pattern index (the same file `ADOPT.md` Tier 4 writes), so new code has a pointer to each foundation guide. Optionally add topic stubs (`testing.md`, `git-workflow.md`, `code-style.md`, `security.md`) per `templates/NEW_PROJECT_PROMPT.md`.

### 4. Install lint configs (TypeScript/JavaScript or Python projects)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sneg55/agent-starter](https://github.com/sneg55/agent-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
