---
trigger: always_on
description: AGENTS_POLICY: gateway-v1.1
---

<!--
AGENTS_POLICY: gateway-v1.1
repo_namespace: .agentplane
default_initiator: ORCHESTRATOR
-->

<!-- ap:fragment id="gateway.agents.purpose.purpose" slot="purpose" mutability="replaceable" -->

# PURPOSE

`AGENTS.md` is the policy gateway for agents in this repository.
It provides strict routing, hard constraints, and command contracts.
Detailed procedures live in canonical modules from `## CANONICAL DOCS`.

<!-- /ap:fragment -->

<!-- ap:fragment id="gateway.agents.purpose.project" slot="purpose" mutability="replaceable" -->

## PROJECT

- Repository type: user project initialized with `agentplane`.
- Gateway role: keep this file compact and deterministic; move scenario-specific details to policy modules.
- CLI rule: use `agentplane` from `PATH`; if unavailable, stop and request installation guidance (do not invent repo-local entrypoints).
- Startup shortcut: run `## COMMANDS -> Preflight`, then use `agentplane quickstart`; activate `agentplane role ORCHESTRATOR` for planning and `agentplane role <ROLE>` for the active owner before owner-scoped execution; then apply `## LOAD RULES` before any mutation. The guarded route is determined by `workflow_mode` in `.agentplane/config.json`; use `agentplane quickstart` as the canonical summary of the active path before mutating. In `branch_pr`, start from `agentplane work start ... --worktree`; in `direct`, stay in the current checkout and use the task lifecycle route.

<!-- /ap:fragment -->
<!-- ap:fragment id="gateway.agents.source_of_truth.sources.of.truth" slot="source_of_truth" mutability="replaceable" -->

## SOURCES OF TRUTH

Priority order (highest first):

1. Enforcement: CI, tests, linters, hooks, CLI validations.
2. Policy gateway: `AGENTS.md`.
3. Canonical policy modules from `## CANONICAL DOCS`.
4. CLI guidance: `agentplane quickstart`, `agentplane role <ROLE>`, `.agentplane/config.json`.
5. Reference examples from `## REFERENCE EXAMPLES`.

Conflict rule:

- If documentation conflicts with enforcement, enforcement wins.
- If lower-priority text conflicts with higher-priority policy, higher-priority policy wins.

<!-- /ap:fragment -->
<!-- ap:fragment id="gateway.agents.hard_constraint.scope.boundary" slot="hard_constraint" mutability="append_only" -->

## SCOPE BOUNDARY

- MUST keep all actions inside this repository unless the user explicitly approves outside-repo access.
- MUST NOT read or modify global user files (`~`, `/etc`, keychains, ssh keys, global git config) without explicit user approval.
- MUST treat network access as approval-gated when `agents.approvals.require_network=true`.

<!-- /ap:fragment -->
<!-- ap:fragment id="gateway.agents.commands.commands" slot="commands" mutability="replaceable" -->

## COMMANDS

### Preflight

```bash
agentplane config show
agentplane quickstart
agentplane task list
git status --short --untracked-files=no
git rev-parse --abbrev-ref HEAD
```

### Task lifecycle

```bash
agentplane task new --title "..." --description "..." --priority med --owner <ROLE> --tag <tag>
agentplane task plan set <task-id> --text "..." --updated-by <ROLE>
agentplane task plan approve <task-id> --by ORCHESTRATOR
agentplane task start-ready <task-id> --author <ROLE> --body "Start: ..."
agentplane verify <task-id> --ok|--rework --by <ROLE> --note "..." [--observation "..." --impact "..." --resolution "..."] [--local-only]
agentplane finish <task-id> --author <ROLE> --body "Verified: ..." --result "..." --commit <git-rev>
```

### branch_pr lifecycle

```bash
agentplane work start <task-id> --agent <ROLE> --slug <slug> --worktree
agentplane pr open <task-id> --branch task/<task-id>/<slug> --author <ROLE>
agentplane pr update <task-id>
agentplane integrate <task-id> --branch task/<task-id>/<slug> --run-verify
agentplane finish <task-id> --author INTEGRATOR --body "Verified: ..." --result "..." --commit <git-rev> --close-commit
```

### Verification

```bash
agentplane task verify-show <task-id>
agentplane verify <task-id> --ok|--rework --by <ROLE> --note "..." [--observation "..." --impact "..." --resolution "..."] [--local-only]
agentplane incidents advise <task-id>
agentplane incidents collect <task-id> --check
agentplane doctor
node .agentplane/policy/check-routing.mjs
```

<!-- /ap:fragment -->
<!-- ap:fragment id="gateway.agents.body.tooling" slot="body" mutability="replaceable" -->

## TOOLING

- Use `## COMMANDS` as the canonical command source.
- Use `agentplane quickstart` as the canonical installed startup path and `agentplane role <ROLE>` to activate the current role before role-scoped planning or execution.
- For policy changes, routing validation MUST pass via `node .agentplane/policy/check-routing.mjs`.

<!-- /ap:fragment -->
<!-- ap:fragment id="gateway.agents.body.shared.prompt.contract" slot="body" mutability="replaceable" -->

## SHARED PROMPT CONTRACT

- Outcome-first, concise, evidence-first: state goal, success criteria, constraints, stop rules, and output; use procedure only for command contracts, state machines, or irreversible gates; ask one narrow question only when missing information changes scope, task graph, security, or irreversible action.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [basilisk-labs/agentplane](https://github.com/basilisk-labs/agentplane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
