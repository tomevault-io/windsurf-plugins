---
trigger: always_on
description: Enables sending custom string messages over the same WebSocket
---

<div align="center">

# AGENTS.md — Agent Guide for the valtio-y Monorepo

</div>

> **Goal:** Give you (the AI agent) a predictable playbook for working inside this Bun workspace. If you ever feel stuck, skim this file first.

---

## 🧭 Quick Start Cheat Sheet

| If you need to…               | Run this command (from root)     |
| ----------------------------- | -------------------------------- |
| Lint code                     | `bun run lint`                   |
| Fix lint issues automatically | `bun run lint:fix`               |
| Format code                   | `bun run format`                 |
| Build all packages            | `bun run build` (uses turbo)     |
| Test all packages             | `bun run test` (uses turbo)      |
| Type check all packages       | `bun run typecheck` (uses turbo) |
| Dev mode all packages         | `bun run dev` (uses turbo)       |
| Create a changeset            | `bun changeset`                  |

Keep these commands nearby—most tasks you perform will be a combination of them.

---

## 🗺️ Table of Contents

1. [Standard Agent Workflow](#agent-workflow)
2. [Guiding Human Developers](#human-workflow)
3. [Git Workflow & Commit Conventions](#git-workflow)
4. [Command Reference Library](#command-reference)
5. [Project Overview](#project-overview)
6. [Critical Rules for AI Agents](#critical-rules)
7. [Documentation Reference](#documentation)
8. [Quick Wins](#quick-wins)
9. [Need Help?](#need-help)

Use the links above to jump directly to the guidance you need.

---

<a id="agent-workflow"></a>

## 🛠️ Standard Agent Workflow

This monorepo is designed so you can verify correctness entirely through static checks and targeted tests.

1. **Execute targeted tests** – Use `cd valtio-y && bun run test` to run all tests, or `bun vitest --run src/path/to/test.ts` to run specific test files. Avoid watch mode unless explicitly requested.

2. **Type check** – `cd valtio-y && bun run typecheck` ensures TypeScript correctness and catches type errors, const reassignments, type mismatches, and other compilation errors. **This is equally important as linting** – both tools serve different purposes.

3. **Lint + Format** – Run `bun run lint:fix` to fix lint issues automatically, then `bun run format` to format code. Both commands are fast enough to run across the whole repo when needed. Use `bun run check` to run both formatting and linting in one command.

4. **Repeat as Needed** – After changes, re-run the relevant steps. If you touch multiple packages, rebuild and verify each affected project.

**Important:** Use **both** type checking and linting in equal parts:

- **Type checking (`bun run typecheck`)** catches type errors, const violations, duplicate declarations
- **Linting (`bun run lint`)** catches code style, patterns, and best practices

These tools are complementary, not redundant. Your IDE may show both types of errors, but you need to run both commands to verify correctness.

This is **your** loop. You do **not** need to run dev servers or browsers to verify your work.

---

<a id="human-workflow"></a>

## 🧑‍🤝‍🧑 Guiding Human Developers

When answering questions from humans, point them to the development workflow:

**For testing:**

```bash
cd valtio-y && bun run test
```

**For development with watch mode:**

```bash
cd valtio-y && bun run dev
```

**For building:**

```bash
cd valtio-y && bun run build
```

**For running examples:**
Each example directory has its own setup. Navigate to the example directory and follow its README or package.json scripts.

---

<a id="git-workflow"></a>

## 🌿 Git Workflow & Commit Conventions

### Branch Naming

Use descriptive branch names that follow this pattern:

```
<type>/<short-description>
```

**Common types:**

- `feat/` - New features
- `fix/` - Bug fixes
- `chore/` - Maintenance tasks (dependencies, tooling, etc.)
- `docs/` - Documentation updates
- `refactor/` - Code refactoring
- `test/` - Test additions or updates
- `perf/` - Performance improvements

**Examples:**

```bash
git checkout -b feat/partykit-provider
git checkout -b fix/array-sync-bug
git checkout -b chore/readme-updates
git checkout -b docs/architecture-guide
```

### Commit Message Convention

We follow [Conventional Commits](https://www.conventionalcommits.org/) format:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**⚠️ IMPORTANT: Scope is REQUIRED. All commits and PR titles must include a scope.**

**Type:**

- `feat` - New feature
- `fix` - Bug fix
- `chore` - Maintenance tasks
- `docs` - Documentation changes
- `refactor` - Code refactoring
- `test` - Test updates
- `perf` - Performance improvements
- `ci` - CI/CD changes

**Scope** (REQUIRED - indicates what part of the codebase):

- `core` - Main valtio-y package
- `docs` - Documentation
- `examples` - Example applications
- `ci` - CI/CD pipeline
- `deps` - Dependency updates
- `tests` - Test infrastructure
- `repo` - Repository-wide changes (tooling, configuration, etc.)

**Examples:**

```bash
feat(core): add Y.Text integration support
fix(core): resolve array sync race condition
chore(docs): clarify provider setup in README
docs(architecture): add data flow diagrams
feat(examples): add PartyKit todo example
fix(ci): update Node version in GitHub Actions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [valtiojs/valtio-y](https://github.com/valtiojs/valtio-y) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
