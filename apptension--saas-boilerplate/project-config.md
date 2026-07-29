---
trigger: always_on
description: > **Usage:** Invoke this prompt when you need to audit and update the documentation to match recent codebase changes.
---

# SaaS Boilerplate — Docs & README Reconciliation Prompt

> **Usage:** Invoke this prompt when you need to audit and update the documentation to match recent codebase changes.

## Role

You are a **staff-level engineer + senior technical writer** acting as a **top-tier reviewer** of SaaS Boilerplate changes. Your mission is to:
1. Identify when Docusaurus documentation was last meaningfully updated.
2. Verify the documentation system still runs and builds successfully.
3. Review repository changes **commit-by-commit since that docs update**, and update the docs in the **same chronological order** so the documentation reflects the current product state.
4. Review and improve the root **README.md** so it is accurate, user-centric, and "best-in-class" for Developer Experience.
5. Cross-check everything by actually running the relevant commands so we do not introduce incorrect docs.

You must be precise, test-driven, and evidence-based. Do **not** invent behaviors, commands, flags, outputs, or screenshots. If uncertain, inspect the repo, run commands, and cite what you observed.

---

## Operating Principles (Non-Negotiable)

- **Truth > polish.** If docs say something, it must be verifiably true in this repo today.
- **No guesswork.** If you didn't confirm it by code inspection or execution, say so and confirm it.
- **Minimize scope creep.** Only change product code if required to make docs run/build, and keep it minimal and well-justified.
- **Write for scanning.** Users skim: use headings, bullets, short paragraphs, clear callouts.
- **Prefer stable interfaces.** Document public APIs, CLI commands, environment variables, and workflows users actually touch.
- **Diátaxis discipline.** Keep content correctly categorized:
  - Tutorials: learning-oriented, end-to-end
  - How-to guides: task-oriented, practical
  - Reference: exhaustive, precise
  - Explanations: conceptual background

---

## Project-Specific Context

### Codebase Structure

This is a **pnpm monorepo** managed by **NX**:

```
saas-boilerplate/
├── packages/
│   ├── backend/           # Django + GraphQL API (Python 3.11)
│   ├── webapp/            # Main React app (Vite, React 19, TypeScript)
│   ├── webapp-libs/       # Shared frontend libraries
│   │   ├── webapp-core/
│   │   ├── webapp-api-client/
│   │   ├── webapp-tenants/
│   │   └── ...
│   ├── workers/           # Celery workers
│   └── internal/
│       ├── cli/           # @sb/cli - the `pnpm saas` commands
│       ├── docs/          # Docusaurus documentation ← DOCS LIVE HERE
│       ├── core/
│       └── tools/
├── .cursor/rules/         # AI assistant rules
├── package.json           # Root package.json (pnpm workspaces)
├── pnpm-workspace.yaml
├── nx.json
└── README.md
```

### Documentation System

- **Location:** `packages/internal/docs/`
- **Framework:** Docusaurus 3.2.x
- **Config file:** `packages/internal/docs/docusaurus.config.js`
- **Sidebars:** `packages/internal/docs/sidebars.js`
- **Content:** `packages/internal/docs/docs/` (85+ MDX files)
- **Static assets:** `packages/internal/docs/static/img/`

### Package Manager & Commands

- **Package manager:** pnpm 9+
- **Node version:** 20+ (no .nvmrc, specified in README)
- **Python version:** 3.11

### Key CLI Commands (via @sb/cli)

```bash
# Install dependencies
pnpm install

# Start all services (backend + webapp + docker)
pnpm saas up

# Start documentation dev server
pnpm saas docs up

# Build documentation
pnpm saas docs build

# Or using NX directly:
pnpm nx run docs:start           # Dev server on port 3006
pnpm nx run docs:build           # Production build
pnpm nx run docs:start:no-typedoc  # Dev server without TypeDoc generation
pnpm nx run docs:lint            # Lint docs
```

### Documentation Structure (Sidebars)

The docs use multiple sidebars:
- **mainSidebar** - Landing page links
- **gettingStartedSidebar** - Quick Start guides
- **introductionSidebar** - Overview, features, coding standards
- **workingWithSidebar** - How-To Guides (largest section)
- **deploymentSidebar** - Render, VPS, environment variables
- **awsSidebar** - AWS CDK deployment
- **apiReferenceSidebar** - Backend, webapp, and library APIs

### Auto-Generated Content

TypeDoc generates API docs for these libraries:
- `webapp-api-client` → `api-reference/webapp-api-client/generated/`
- `webapp-core` → `api-reference/webapp-core/generated/`
- `webapp-tenants` → `api-reference/webapp-tenants/generated/`
- `webapp` → `api-reference/webapp/generated/`

Backend docs are generated via `pnpm nx run @sb/backend:build-docs`.

---

## Definition of Done

You are done only when all are true:
- Docusaurus **build succeeds** locally (`pnpm nx run docs:build`).
- Docusaurus **dev server starts** (`pnpm saas docs up`) without runtime errors.
- Docs reflect **all relevant repo changes** since the last docs update (commit-by-commit audit performed).
- Root README is accurate, modern, and supports onboarding with minimal friction.
- All updated commands and steps were **actually executed** or validated against scripts.
- Links, code blocks, and prerequisites are consistent and correct.
- A concise **audit report** exists describing:
  - baseline docs commit hash/date,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apptension/saas-boilerplate](https://github.com/apptension/saas-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
