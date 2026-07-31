---
trigger: always_on
description: AGENTS_POLICY: gateway-v1.1
---

<!--
AGENTS_POLICY: gateway-v1.1
repo_namespace: .agentplane
default_initiator: ORCHESTRATOR
-->


# PURPOSE

`AGENTS.md` is the policy gateway for agents in this repository.
It provides strict routing, hard constraints, and command contracts.
Detailed procedures live in canonical modules from `## CANONICAL DOCS`.



## PROJECT

- Repository type: user project initialized with `agentplane`.
- CLI rule: prefer `ap` for compact agent-oriented commands; fall back to `agentplane`; if neither is available, stop and request installation guidance (do not invent repo-local entrypoints).
- Startup shortcut: run `## COMMANDS -> Preflight`; use `ap quickstart`; activate `ap role ORCHESTRATOR` for planning and `ap role <ROLE>` for execution; then apply `## LOAD RULES` before mutation. The guarded route is determined by `workflow.mode` in `.agentplane/WORKFLOW.md`; treat `ap task brief <task-id>` and `ap task next-action <task-id> --explain` as the route oracle: follow the emitted checkout, blocker, and next command instead of reconstructing workflow state.


## SOURCES OF TRUTH

Priority order (highest first):

1. Enforcement: CI, tests, linters, hooks, CLI validations.
2. Policy gateway: `AGENTS.md`.
3. Canonical policy modules from `## CANONICAL DOCS`.
4. CLI guidance: `ap quickstart`, `ap role <ROLE>`, `.agentplane/WORKFLOW.md`.
5. Reference examples from `## REFERENCE EXAMPLES`.

Conflict rule:

- If documentation conflicts with enforcement, enforcement wins.
- If lower-priority text conflicts with higher-priority policy, higher-priority policy wins.


## SCOPE BOUNDARY

- MUST keep all actions inside this repository unless the user explicitly approves outside-repo access.
- MUST NOT read or modify global user files (`~`, `/etc`, keychains, ssh keys, global git config) without explicit user approval.
- MUST treat network access as approval-gated when `agents.approvals.require_network=true`.


## COMMANDS

### Preflight

```bash
ap config show
ap quickstart
ap task list
ap task active
git status --short --untracked-files=no
git status --short --untracked-files=all
git rev-parse --abbrev-ref HEAD
```

### Route commands

```bash
ap task brief <task-id>
ap task next-action <task-id> --explain
ap work resume <task-id>
```

### Task lifecycle

```bash
ap task new --title "..." --description "..." --priority med --owner <ROLE> --tag <tag>
ap task plan set <task-id> --text "..." --updated-by <ROLE>
ap task plan approve <task-id> --by ORCHESTRATOR
ap task start-ready <task-id> --author <ROLE> --body "Start: ..."
ap verify <task-id> --ok|--rework --by <ROLE> --note "..." [--observation "..." --impact "..." --resolution "..."] [--local-only]
ap finish <task-id> --author <ROLE> --body "Verified: ..." --result "..." --commit <git-rev>
```

### branch_pr lifecycle

```bash
ap work start <task-id> --agent <ROLE> --slug <slug> --worktree
ap task start-ready <task-id> --author <ROLE> --body "Start: ..."
git commit -m "Implement <task>"
ap task verify-show <task-id>
ap pr open <task-id> --branch task/<task-id>/<slug> --author <ROLE>
ap verify <task-id> --ok|--rework --by <ROLE> --note "..."
ap evaluator run <task-id> --verdict pass|rework|blocked|human_review --summary "..." --finding "..." --evidence <path-or-check>
ap integrate <task-id> --branch task/<task-id>/<slug> --run-verify
ap finish <task-id> --author INTEGRATOR --body "Verified: ..." --result "..." --commit <git-rev> --close-commit
```

### Verification

```bash
ap vshow <task-id>
ap verify <task-id> --ok|--rework --by <ROLE> --note "..." [--observation "..." --impact "..." --resolution "..."] [--local-only]
ap evaluator run <task-id> --verdict pass|rework|blocked|human_review --summary "..." --finding "..." --evidence <path-or-check> [--missing-test "..." --hidden-assumption "..." --residual-risk "..."]
ap incidents advise <task-id>
ap incidents collect <task-id> --check
ap doctor
node .agentplane/policy/check-routing.mjs
```


## TOOLING

- Use `## COMMANDS` as the canonical command source.
- Use `ap quickstart` as the compact installed startup path and `ap role <ROLE>` to activate the current role before role-scoped planning or execution.
- For policy changes, routing validation MUST pass via `node .agentplane/policy/check-routing.mjs`.


## SHARED PROMPT CONTRACT

- Outcome-first, concise, evidence-first: state goal, success criteria, constraints, stop rules, and output; use procedure only for command contracts, state machines, or irreversible gates.
- Ambiguity rule: ask one narrow question only when missing information changes scope, security, task graph, or irreversible action; otherwise act under stated assumptions.
- Route/persistence rule: for multi-step or tool-heavy work, send a short preamble, load `ap task brief <task-id>`, follow `ap task next-action <task-id> --explain`, and persist through implementation + verification unless blocked.
- Context rule: load only matched policy, task README, Verify Steps, and relevant files; never cache mutable task state; final output names actions, checks, blockers/drift, and next approval.


IF `.agentplane/user-instructions.md` exists THEN LOAD it as `gateway.user.instructions`.


## LOAD RULES

Routing is strict. Load only modules that match the current task.

### Always imports for mutating tasks


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Comamoca/subaru](https://github.com/Comamoca/subaru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
