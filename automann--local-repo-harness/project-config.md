---
trigger: always_on
description: routes local-repo-harness requests through the CLI and hook automation plugin for init, update, scaffold, migrate, audit, repair, and ship workflows
---


# local-repo-harness Skill Router

`local-repo-harness` is the CLI and hook automation package for repo-local
agentic development. The `repo-harness` skill entrypoint is a router over the
versioned workflow engine and command facades.

Compatibility boundary:

- internal engine: CLI plus hook-backed tasks-first harness
- contract ID: tasks-first-harness-v1
- canonical skill name: `repo-harness`
- canonical CLI and package name: `local-repo-harness`
- command facade skill slugs: `repo-harness-*`
- retired legacy aliases: `repo-harness-skill`, `project-initializer`
- new-project creation surface: `repo-harness-scaffold` (secondary generator)

`repo-harness-skill` and `project-initializer` are retired: do not create,
sync, or search their directories under `~/.codex/skills` or
`~/.claude/skills` as active upstream roots; installed-copy sync removes any
existing copies. Historical `project-initializer` markers in generated files
may remain only as legacy evidence.

The skill should not carry the whole workflow contract in prose. It should:

1. inspect the repository
2. classify the workflow state
3. choose the correct path
4. rely on the repo contract, migration scripts, and tests for enforcement

## When to use

- install or refresh the CLI+hooks workflow in an existing repo
- create a new project or module scaffold only when the user asks for a new product/module skeleton, then attach the harness
- migrate an older repo to the current tasks-first harness
- audit drift between prompts, hooks, scripts, and repo-local contract files
- repair broken task-sync, workflow-contract, or handoff surfaces

## When not to use

- runtime bug debugging inside an already healthy AI workflow
- generic project scaffolding unrelated to AI routing or repo-local workflow contracts
- using scaffold to adopt an existing repo; route that to `repo-harness-init`,
  `repo-harness-migrate`, `repo-harness-upgrade`, or `repo-harness-repair`
- ordinary product feature work

## Router Protocol

Always start with structured inspection, not prompt guessing.

### Step 1. Inspect first

Run:

- `bun scripts/inspect-project-state.ts --repo <path> --format text`
  - fallback: `node --experimental-strip-types scripts/inspect-project-state.ts --repo <path> --format text`

Read the result fields:

- `mode`
- `legacy_contract_version`
- `drift_signals`
- `required_decisions`
- `safe_defaults`

### Step 2. Choose one path

If the request maps to a public command facade, name that route before running
checks or edits, then read the matching
`assets/skill-commands/<repo-harness-command>/SKILL.md` and follow that
facade's protocol. For example, pre-merge or release readiness requests route
to `repo-harness-check`, while broken task sync, hook routing, handoff, context,
policy, or helper surfaces route to `repo-harness-repair`.

1. **Scaffold**
   - use only when creating a new project, app, or module skeleton
   - route to `repo-harness-scaffold`
   - choose the A-K project catalog entry, then optional `ai_native_profile`
   - attach the harness after the project structure exists
   - do not use this path for existing-repo adoption
2. **Initialize**
   - use when the repo has no meaningful tasks-first workflow yet
3. **Migrate**
   - use when the repo has legacy workflow docs, missing contract manifest, or stale harness artifacts
4. **Audit**
   - use when the repo mostly works but the user wants drift analysis and enforcement review
5. **Repair**
   - use when the repo has a current contract surface but broken task-sync, hooks, or handoff behavior
### Step 3. Prefer engine actions over prompt-only fixes

Default order:

1. for new projects, scaffold the requested project/module shape first
2. migrate legacy docs if needed
3. install or refresh workflow contract artifacts
4. sync hooks, helpers, and templates
5. merge the guidance-only `external_tooling` profile into `.ai/harness/policy.json`
6. verify the repo-local contract

Do not treat hooks as the primary source of truth. The repo contract lives in repo files.

## Core Engine Surfaces

The single machine-readable contract source is:

- `assets/workflow-contract.v1.json`

The installed runtime copy inside a repo is:

- `.ai/harness/workflow-contract.json`

The main engine entrypoints are:

- `scripts/inspect-project-state.ts`
- `scripts/migrate-workflow-docs.ts`
- `scripts/migrate-project-template.sh`
- `scripts/check-agent-tooling.sh`
- `scripts/check-task-workflow.sh`
- `scripts/create-project-dirs.sh`

## CLI Command Facade Surface

The command facades live in `assets/skill-commands/` as compatibility wrappers
over the same CLI and hook engine. Use them for routing when the host discovers
skills; the implementation authority stays in the CLI, scripts, hooks, and
contract files:

- `repo-harness-plan`: interactive planning; no repo mutation by default
- `repo-harness-review`: plan review across product, engineering, design, and DevEx
- `repo-harness-autoplan`: automatic plan -> self-review twice -> implementation -> check -> ship pipeline
- `repo-harness-ship`: validate finished worktrees, push branches, and create PRs by default

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [automann/local-repo-harness](https://github.com/automann/local-repo-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
