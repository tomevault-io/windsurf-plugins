---
trigger: always_on
description: > **SYSTEM NOTICE**: This is the Single Source of Truth (SSOT) for all AI agents (Gemini, Claude, Open Code) working on the `virtual-module-core` project.
---

# AGENTS.md

> **SYSTEM NOTICE**: This is the Single Source of Truth (SSOT) for all AI agents (Gemini, Claude, Open Code) working on the `virtual-module-core` project.

## 1. Identity & Configuration Resolution

Before executing any task, identify your runtime environment and load the corresponding configuration:

| Agent Identity  | Skills Path      | Config Dir  | Instruction Source           |
| :-------------- | :--------------- | :---------- | :--------------------------- |
| **Antigravity** | `.agent/skils/`  | `.agents/`    | `AGENTS.md` (Native)         |

**Rule**: Whenever the documentation below refers to `{COMMAND_DIR}`, substitute it with the **Command Path** specific to your identity above.

---

## 2. Overview

This repository implements a **Spec-Driven Development (SDD)** workflow system. The workflow guides feature development through structured phases: constiution -> specification -> clarify -> checlist -> planning -> analyze -> task generation -> tasks to issues -> implementation.
                      ^            |
                      |            |
                      --------------

## 3. Workflow Commands

All workflow commands are defined in `{COMMAND_DIR}` and execute via the `/command-name` syntax.

### Feature Development Lifecycle

1. **`/speckit-constitution [principles]`** - Create or update project constitution (`.specify/memory/constitution.md`)
   - Defines non-negotiable development principles
   - Templates are synchronized automatically
   - Uses semantic versioning (MAJOR.MINOR.PATCH)

2. **`/speckit-specify <feature description>`** - Create feature specification
   - Generates `specs/###-feature/spec.md` from template
   - Focus: WHAT and WHY (business requirements, user scenarios)
   - Avoid: HOW (no tech stack, APIs, implementation details)

3. **`/speckit-clarify`** - Resolve specification ambiguities (run BEFORE `/speckit.plan`)
   - Asks up to 5 targeted clarification questions
   - Updates spec.md with answers in `## Clarifications` section
   - Reduces rework during implementation

4. **`/speckit-plan [context]`** - Generate implementation plan
   - Requires completed spec.md
   - Creates: `research.md`, `data-model.md`, `contracts/`, `quickstart.md`, agent-specific guidance
   - Validates against constitution principles
   - Stops before task generation (use `/speckit.tasks` next)

6. **`/speckit.analyze`** - Cross-artifact consistency analysis (run AFTER `/speckit.tasks`)
   - Read-only validation across spec.md, plan.md, tasks.md
   - Detects: duplications, ambiguities, coverage gaps, constitution violations
   - Severity: CRITICAL, HIGH, MEDIUM, LOW
   - Provides remediation suggestions (does NOT auto-fix)

5. **`/speckit.tasks [context]`** - Generate actionable task breakdown
   - Requires completed plan.md
   - Creates dependency-ordered `tasks.md`
   - Tasks marked `[P]` can run in parallel
   - Sequential tasks must run in order

7. **`/speckit.implement`** - Execute implementation from tasks.md
   - Phase-by-phase execution: Setup → Tests → Core → Integration → Polish
   - Respects task dependencies and parallel markers
   - Marks completed tasks with `[X]` in tasks.md

## Repository Structure

```

.specify/
├── memory/
│   └── constitution.md    \# Project constitution (template)
├── scripts/bash/          \# Helper scripts for workflow commands
│   ├── check-prerequisites.sh
│   ├── create-new-feature.sh
│   ├── setup-plan.sh
│   └── common.sh
└── templates/             \# Templates for generated artifacts
    ├── spec-template.md
    ├── plan-template.md
    ├── tasks-template.md
    └── agent-file-template.md

specs/\#-feature/         \# Generated per-feature (created by workflow)
├── spec.md
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
└── tasks.md

```

## Key Workflow Scripts

All scripts must be run from repository root.

### check-prerequisites.sh

```bash
.specify/scripts/bash/check-prerequisites.sh --json
.specify/scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
.specify/scripts/bash/check-prerequisites.sh --json --paths-only
```

- Returns: `FEATURE_DIR`, `AVAILABLE_DOCS`, `BRANCH`, `REPO_ROOT`
- Use `--require-tasks` for implementation phase
- Use `--paths-only` for minimal output

### create-new-feature.sh

```bash
.specify/scripts/bash/create-new-feature.sh --json "feature description"
```

- Creates feature branch and initializes spec file
- Returns: `BRANCH_NAME`, `SPEC_FILE`

### setup-plan.sh

```bash
.specify/scripts/bash/setup-plan.sh --json
```

- Returns: `FEATURE_SPEC`, `IMPL_PLAN`, `SPECS_DIR`, `BRANCH`

## Development Principles

### Constitution Authority

- The constitution (`.specify/memory/constitution.md`) is **non-negotiable**
- All design artifacts must validate against constitution principles
- Violations are flagged as CRITICAL during `/speckit.analyze`

### Test-Driven Development

- Tasks follow TDD: Tests written → User approved → Tests fail → Implement
- Test tasks appear before implementation tasks in tasks.md

### Artifact Separation

- **spec.md**: Business requirements (non-technical stakeholders)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reidlai/virtual-module-core](https://github.com/reidlai/virtual-module-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
