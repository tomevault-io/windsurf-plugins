---
trigger: always_on
description: This repository uses a modular instruction system to guide AI coding agents. It consists of several files, each focusing on specific aspects:
---

# AI Coding Guidelines

This repository uses a modular instruction system to guide AI coding agents. It consists of several files, each focusing on specific aspects:

- **LANGUAGE.md** – Language-specific style, libraries, and testing conventions (for Go).
- **WORKFLOW.md** – Workflow and project management rules (how to plan, track tasks, commit, and test).
- **PLANS.md** – ExecPlan specification (how to create and use execution plans for complex tasks).
- **APP.md** – Application-specific architecture and design info for codex-sdk-go.

**Always follow all applicable instructions.** If guidelines appear to conflict, the order of precedence is defined under **Precedence Rules** below.

## Precedence Rules

When multiple instruction files apply, the more specific rules override more general ones:

1. **Active ExecPlan:** If an ExecPlan is in progress for the current task/feature, follow the plan's steps and decisions first and foremost. (ExecPlans are detailed design docs for a specific epic/feature, see PLANS.md.)
2. **Application-Specific (APP.md):** Project-specific architecture or conventions unique to this app take priority next.
3. **Language-Specific (LANGUAGE.md):** General coding style and best practices for Go, unless overridden by app-specific needs above.
4. **Workflow & Process (WORKFLOW.md):** Project workflow rules (e.g. issue tracking, commit process, testing workflow) are next – these typically don’t conflict with coding style but govern process.
5. **Core Guidelines (AGENTS.md):** The base rules in this file and any global defaults apply where others do not specify.

If any confusion arises, prefer the instruction source closest to the problem (for example, a rule in APP.md tailored to the module you are working on outweighs a generic language rule). When in doubt, err on the side of clarity: update the relevant instruction file or ask for guidance rather than guessing.

## ExecPlans

For **complex features or significant refactors**, you **must create and use an ExecPlan**. An ExecPlan is a detailed, step-by-step design and execution plan that guides work from design through implementation. Specifically:

- **When to use:** If the task corresponds to a Linear **epic** (or equivalent large user story) or if the problem is non-trivial (multiple steps, uncertainty, or cross-cutting concerns), start by writing an ExecPlan. (See WORKFLOW.md for criteria and Linear usage.)
- **How to use:** Before coding, create a new Markdown file under `plans/` (e.g. `plans/CDX-4.md`) and copy the ExecPlan skeleton from PLANS.md. Fill it out with research, decisions, and planned steps. Obtain confirmation (if required) on the plan, then implement according to the plan.
- **During implementation:** Follow the ExecPlan “to the letter”. Update the plan as you make progress, discover surprises, or change approach. **Do not skip updating the plan.** The sections like Progress, Decision Log, etc., must remain current.
- **After completion:** Ensure the ExecPlan’s outcomes section is filled and it stands as an accurate record of what was done and why. Future contributors should be able to read it and understand the feature.

By using ExecPlans, we ensure large tasks are well-scoped and tracked. *If an ExecPlan is required but not present, stop and create one before proceeding.* (This prevents tackling complex work in an ad-hoc way.)

## AI Agent Conduct (Codex Contract)

These are explicit **Do’s and Don’ts** the AI agent must follow at all times:

**DO:**
- **Follow the Plan & Workflow:** Adhere strictly to the ExecPlan steps when one is in use. For smaller tasks without a plan, still break down the work logically and follow project workflow rules.
- **Track all work in Linear:** Assume that every significant code change corresponds to a tracked issue or task. If you identify work that isn’t tracked, pause and either create a task (if allowed) or alert the user. Untracked work is not allowed.
- **Write Tests for New Code:** Whenever you implement new functionality or fix a bug, write or update unit tests and integration tests as applicable. Aim for test-driven development: e.g. write failing tests first for new features or bug fixes, then write code to make them pass (especially for complex logic).
- **Document Generated Protocol APIs:** Every exported type, alias, constant, variable, function, and method emitted into `protocol/*_gen.go` must have a GoDoc comment beginning with its exact exported name. Enforce this in `internal/codegen` and its tests; never repair generated comments by hand.
- **Run and Pass Tests:** Run the test suite locally (`go test ./...`) after changes. Ensure all tests pass (and new tests fail before the fix and pass after). Do not consider a task complete until tests are green.
- **Commit Frequently with References:** Make small, logical commits. Each commit message should be in the imperative mood (for example, "Add X", "Fix Y bug") and, when possible, include the tracker ID (for example, "CDX-6: Implement feature X") to tie commits to tasks. Commit only code that compiles and passes tests.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pmenglund/codex-sdk-go](https://github.com/pmenglund/codex-sdk-go) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
