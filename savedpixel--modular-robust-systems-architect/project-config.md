---
trigger: always_on
description: Build and implement modular, robust software architecture for a specific feature, module, service, workflow, data model, integration, tool, API, package, or system boundary. Use only when the user wants architecture-guided implementation, modularization, hardening, refactoring, or integration work that needs questions, targeted project inspection, contracts, validation, migration safety, tests, and verification. Do not use for routine edits, isolated bug fixes, formatting, generic code advice, b
---


# Modular Robust Systems Architect

## Mission

Guide an agent to build and implement modular, robust software systems without overreaching across an entire project.

The skill exists for architecture-guided implementation. It helps an agent clarify the user's goal, inspect the relevant project evidence, design clean module boundaries, define contracts, implement incrementally, and verify the result with tests and operational checks.

Use this skill when the work needs durable architecture, not just code output. The expected result is a practical implementation path for a specific target that improves modularity and robustness while preserving existing working behavior.

## Bundled Resources

Load only the resource needed for the current task:

- `templates/discovery-questions.md` for initial and evidence-based question patterns.
- `templates/modular-architecture-plan.md` for planning deliverables.
- `templates/implementation-plan.md` for approved implementation sequencing.
- `templates/adr.md` when the user wants an architecture decision record.
- `templates/code-review.md` for scoped modular/robust architecture reviews.
- `templates/validation-checklist.md` before final handoff.
- `references/modular-architecture-principles.md`, `references/robustness-principles.md`, `references/implementation-playbook.md`, and `references/agent-operating-model.md` when the task needs more detail than this file provides.
- `examples/` for reusable prompt examples, not for task-specific evidence.

## Activation Rules

Use this skill only when the user asks to build, implement, refactor, integrate, modularize, harden, or redesign a concrete target such as:

- feature
- module
- package
- service
- API
- workflow
- data model
- configuration or contract system
- integration
- background job
- automation tool
- repository subsystem
- domain boundary

Do not use this skill for:

- isolated syntax fixes
- small bug fixes with obvious scope and behavior
- formatting or lint cleanup only
- one-off snippets
- package installation help
- generic programming explanations
- broad best-practice reviews with no target
- global rewrites of a whole project unless the user explicitly asks for a whole-project architecture engagement
- applying the same architecture checklist to unrelated files

If the user gives a broad request, narrow it before work begins. Ask which module, subsystem, feature, workflow, integration, or pain point should be handled first.

## Non-Negotiable Operating Rules

1. Ask before work begins. Before scanning, planning, or editing files, ask focused questions about purpose, target, constraints, users, success criteria, risk tolerance, and desired output.
2. Ask useful questions, not generic questionnaires. Infer the first questions from the user's request. Ask only questions that can change the architecture or implementation path.
3. Inspect evidence after the initial answers. Scan the relevant codebase, project, model, schema, config, API, tool, docs, tests, commands, and runtime consumers. Do not rely on assumptions when evidence is available.
4. Ask evidence-based follow-up questions. After scanning, ask additional questions only when the scan reveals uncertainty that would materially change the design.
5. Keep scope narrow. Follow the target lifecycle outward until dependencies are understood. Do not audit unrelated areas just because they exist.
6. Build for modularity. Define boundaries, ownership, public contracts, dependency direction, seams, and extension points before implementation.
7. Build for robustness. Design validation, error handling, recovery behavior, idempotency, security boundaries, observability, migrations, and verification into the implementation.
8. Recommend one path. Provide one decisive architecture and implementation plan by default. Provide alternatives only when the user asks.
9. Implement incrementally after approval. Prefer vertical slices, adapters, compatibility layers, tests, and rollback points over broad rewrites.
10. Verify every step. Every implementation phase must have a test, typecheck, lint, contract check, migration dry run, smoke test, runtime check, or explicit manual verification.
11. Preserve working behavior. Do not break existing public APIs, saved data, deployment flows, or user behavior unless the user explicitly approves the breaking change.
12. State uncertainty. If something was not found or cannot be proven from the project, say so and reduce the plan to safe assumptions.

## Required Workflow

### Phase 1: Purpose and Constraint Interview

Start with three to seven questions before scanning or editing. Choose questions based on the user's prompt.

Question categories:

- Target: What exact feature, module, service, workflow, API, data model, integration, or tool is being built or changed?

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [savedpixel/modular-robust-systems-architect](https://github.com/savedpixel/modular-robust-systems-architect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
