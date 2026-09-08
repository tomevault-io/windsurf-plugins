---
trigger: always_on
description: This guide applies to the Autryn repository root and all of its subdirectories. It defines how Coding Agents should reason, work, and operate safely. Treat source code, tests, and configuration as the authority for project facts. Start at the [Wiki index](.wiki/index.md) for architecture, runtime behavior, and engineering knowledge.
---

# Autryn Agent Guide

This guide applies to the Autryn repository root and all of its subdirectories. It defines how Coding Agents should reason, work, and operate safely. Treat source code, tests, and configuration as the authority for project facts. Start at the [Wiki index](.wiki/index.md) for architecture, runtime behavior, and engineering knowledge.

Autryn automatically loads `AGENTS.md` from the project root as repository guidance.

## 1. First principles

- Establish the objective, constraints, inputs, outputs, and acceptance evidence before choosing an implementation.
- Begin with the required outcomes and invariants. Decompose the problem into verifiable facts, causal relationships, and necessary conditions instead of inferring conclusions from filenames, surface symptoms, or existing practices.
- Distinguish confirmed facts, reasonable inferences, and unknowns. Use source code, tests, configuration, or observed runtime behavior when they can resolve a question.
- Address root causes. Do not conceal problems by swallowing errors, weakening types, deleting assertions, bypassing approval, or adding special cases.
- Choose the smallest sufficient change that satisfies the objective. Avoid features, abstractions, configuration, and extension points unsupported by requirements or evidence.
- Be able to explain the problem, evidence, trade-offs, risks, and verification behind any significant decision.

## 2. Gathering context

1. Inspect the working tree to identify existing user changes and the boundary of the current task.
2. Use the [Wiki index](.wiki/index.md) to identify the relevant module, Workflow, Interface, or engineering area.
3. Read the relevant Wiki page and use its `sources` field to locate the smallest authoritative scope.
4. Read the target implementation, direct dependencies, corresponding tests, and relevant configuration. Expand further only when call relationships or risk require it.

The README serves users, this Agent guide governs working practices, the Wiki organizes durable knowledge, and Git preserves history. If documentation and implementation disagree, verify current behavior and the intended contract, then correct the obsolete side directly.

## 3. Facts and verification

- Keep static inspection, automated checks, actual execution, and observation of external environments distinct. One kind of evidence cannot substitute for another.
- Never report an unexecuted command, untested platform, or unobserved runtime result as passing.
- Derive project commands from the current `package.json`, scripts, and configuration rather than assuming that a remembered command exists.
- Do not treat `dist/`, `node_modules/`, caches, temporary files, or local runtime state as authoritative source facts.
- Match the scope of verification to the risk of the change. A local check cannot establish a cross-module, cross-platform, or release-level claim.

## 4. Scope and change discipline

- Address only the current task. Preserve existing user changes and avoid incidental refactors, bulk renames, or unrelated formatting.
- Treat answering, reviewing, and diagnosing as read-only by default. Modify files only when the user requests changes or the task explicitly includes implementation.
- A documentation task does not authorize changes to application code, configuration, dependencies, or external systems. An implementation task authorizes only the impact necessary to achieve its objective.
- Before changing anything, identify ownership, dependency direction, Public API, persistence, security, and cross-platform implications.
- Reuse existing domain abstractions. Do not duplicate business logic in the CLI, TUI, tests, or Providers.
- Optimize code for clarity, correctness, and maintainability. Concision does not mean compressed expression. Split or combine functions and files according to single responsibility, cohesion, and independent verifiability.
- Follow the established style of the target file. Use kebab-case for files and directories, PascalCase for types, camelCase for functions and variables, and `import type` for type-only imports.
- Preserve common English terms such as Agent, Runtime, Execution, Provider, Session, Turn, Tool, Skill, Middleware, Delegation, CLI, and TUI.
- Account for Windows, macOS, and Linux in paths, Shell behavior, user Home directories, and binary logic.

## 5. Protected boundaries

The following areas are stable or security-sensitive boundaries. Read the linked Wiki pages, source code, and tests before modifying them:

- Module responsibilities and dependency direction: [System architecture](.wiki/architecture/system.md).
- Agent, Tool, and Delegation: [Agent execution](.wiki/workflows/agent-execution.md), [Tool execution](.wiki/workflows/tool-execution.md), and [Delegation](.wiki/workflows/delegation.md).
- Session and Model behavior: [Session lifecycle](.wiki/workflows/session-lifecycle.md) and [Model selection](.wiki/workflows/model-selection.md).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zijie1024/autryn](https://github.com/zijie1024/autryn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
