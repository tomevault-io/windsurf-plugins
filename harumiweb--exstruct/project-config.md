---
trigger: always_on
description: This repository is organized around the following top-level directories:
---

# ExStruct AI Agents Guide

## 0. Overview

This repository is organized around the following top-level directories:

```text
exstruct/
|- src/           # Main library and implementation code
|- tests/         # Automated tests
|- sample/        # Sample workbooks and example inputs
|- schemas/       # JSON schemas and validation-related assets
|- scripts/       # Utility and maintenance scripts
|- benchmark/     # Benchmark code and performance measurements
|- docs/          # User-facing documentation
|- dev-docs/      # All developer-facing documentation
|- tasks/         # Temporary task notes and working files
|- drafts/        # Draft documents and work-in-progress materials
|- dist/          # Build artifacts and packaged outputs
`- site/          # Generated documentation site output
```

For internal development guidance, architecture notes, ADRs, specifications, and testing references, use `dev-docs/` as the canonical location. Developer-facing documentation should be written there rather than scattered across the repository.

## 1. Workflow Design

### 1. Use Plan mode by default

- Always start tasks with 3 or more steps, or tasks that affect architecture, in Plan mode
- If things stop going well partway through, do not force it; stop immediately and replan
- Use Plan mode not only for implementation, but also for verification steps
- Write detailed specifications before implementation to reduce ambiguity

### 2. Multi-Agent Strategy

- Actively use sub-agents to keep the main context window clean
- Delegate research, investigation, and parallel analysis to sub-agents
- For complex problems, use sub-agents to apply more compute resources
- To keep execution focused, assign one task per sub-agent
- Use explorer for read-heavy codebase exploration
- Use worker for implementation and fixes
- Use reviewer for reviews

### 3. Self-Improvement Loop

- Whenever you receive a correction from the user, record that pattern in `tasks/lessons.md`
- Write rules for yourself so you do not repeat the same mistake
- Keep improving those rules thoroughly until the error rate goes down
- At the start of each session, review the lessons relevant to the project

### 4. Always verify before completion

- Do not mark a task as complete until you can prove that it works
- Compare the main branch and your changes when necessary
- Ask yourself, "Would a staff engineer approve this?"
- Run tests, review logs, and show that it works correctly

### 5. Pursue elegance (with balance)

- Before making an important change, pause and ask, "Is there a more elegant way to do this?"
- If a fix feels hacky, think, "Based on everything I know now, implement an elegant solution"
- Skip this process for simple and obvious fixes (do not over-engineer)
- Question your own work before presenting it

### 6. Autonomous bug fixing

- When you receive a bug report, fix it directly without needing step-by-step guidance
- Use logs, errors, and failing tests to solve it yourself
- Eliminate context switching for the user
- Even without being asked, go fix failing CI tests

---

## 2. Areas Outside the AI's Responsibility (Handled by Humans)

The AI does not own the following areas. Humans make these decisions.

- Specification decisions (the direction of ExStruct's evolution)
- Public API design (deciding whether something is a breaking change)
- Large-scale reorganization of the directory structure
- Security and licensing decisions

However, the AI **may make proposals**.

---

## 3. Required Work Procedure

The AI must always follow the steps below before generating code.

1. **Understand requirements**: Read specifications and design materials, and fully understand the requirements
2. **Consider the design**: Consider function decomposition and model design as needed.
3. **Define the specification**: Based on the requirements, define function argument and return types in `tasks/feature_spec.md`.
4. **Assign tasks**: Clearly define each task and determine the implementation order.
5. **Implement code**: Implement the code while following the standards above.
6. **Review code**: Self-review generated code and confirm that it meets the quality standards.
7. **Generate tests**: Generate test code as needed.
8. **Run tests**: Run the generated test code and confirm that it behaves as expected.
9. **Static analysis**: Run `uv run task precommit-run` and confirm that there are no mypy / Ruff errors.
10. **Update documentation**: If there are changes, update the related documentation as well.

---

## 4. Task Management

1. **Plan first**: Write the plan in `tasks/todo.md` as checkable items
2. **Review the plan**: Review it before starting implementation
3. **Track progress**: Mark completed items as you go
4. **Explain changes**: Provide a high-level summary at each step
5. **Document results**: Add a Review section to `tasks/todo.md`
6. **Record lessons**: Update `tasks/lessons.md` after receiving corrections

---

## 5. Documentation Retention Policy

### Separation of Roles

- `tasks/todo.md` may temporarily hold not only session-specific progress tracking, but also verification results, unresolved items, and summaries of decision rationale.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harumiWeb/exstruct](https://github.com/harumiWeb/exstruct) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
