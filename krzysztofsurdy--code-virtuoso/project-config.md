---
trigger: always_on
description: Agents are sub-agents -- standalone markdown files that define a specialized role, its allowed tools, and its system prompt. Each agent file contains YAML frontmatter for configuration and a markdown body that serves as the agent's system prompt.
---

# Agents

@CONTRIBUTING.md

Agents are sub-agents -- standalone markdown files that define a specialized role, its allowed tools, and its system prompt. Each agent file contains YAML frontmatter for configuration and a markdown body that serves as the agent's system prompt.

Where **skills** provide reference knowledge (patterns, checklists, component docs), **agents** are the actors that use that knowledge to perform work. A skill is a book on the shelf; an agent is the specialist who reads the book and applies it. Delegate to an agent when a task is well-scoped enough that a focused sub-agent will do it better than the orchestrating model switching context mid-conversation.

## Architecture

Agents follow a two-tier model:

**Specialist agents** handle a single, repeatable task type. They are typically read-only, stateless, and operate on whatever the caller hands them. Think of them as focused tools: investigate this, review that, scan for smells.

**Role agents** embody a team position. They carry a broader mandate, preload relevant skills, and some maintain persistent project-level memory across sessions. They own a domain ("the what", "the how", "the when") rather than a single task.

| Aspect | Specialist | Role |
|--------|-----------|------|
| Scope | Single task type | Entire domain of responsibility |
| Skills | 1-4 relevant knowledge skills | Role skill + relevant knowledge skills |
| Memory | Stateless | Some use persistent memory for cross-session context |
| File modification | Mostly read-only (except Doc Writer and Implementer) | Dev roles (Backend, Frontend) write code; others are read-only |
| Isolation | Implementer uses worktree; others run in-place | Backend Dev and Frontend Dev use worktree; others run in-place |

## Specialist Agents

| Agent | Tools | Isolation | Purpose |
|-------|-------|-----------|---------|
| [Investigator](agents/investigator.md) | File reading, code search, shell | -- | Deep codebase exploration, dependency mapping |
| [Implementer](agents/implementer.md) | All | worktree | TDD red-green-refactor execution |
| [Reviewer](agents/reviewer.md) | File reading, code search, shell | -- | Structured code review (SOLID, OWASP, smells) |
| [Refactor Scout](agents/refactor-scout.md) | File reading, code search, shell | -- | Code smell scanning, complexity hotspots |
| [Dependency Auditor](agents/dependency-auditor.md) | Shell, file reading, code search | -- | CVE checks, outdated packages, license audit |
| [Doc Writer](agents/doc-writer.md) | File reading, code search, shell, file editing | -- | Changelogs, API docs, migration guides |
| [Migration Planner](agents/migration-planner.md) | File reading, code search, shell | -- | Migration safety analysis, rollback paths |
| [Test Gap Analyzer](agents/test-gap-analyzer.md) | File reading, code search, shell | -- | Missing test coverage, untested edge cases |
| [Cold Reviewer](agents/cold-reviewer.md) | File reading, code search, shell | -- | Zero-context code review, fresh-eyes findings |
| [Acceptance Verifier](agents/acceptance-verifier.md) | File reading, code search, shell | -- | Spec compliance checking, criteria coverage matrix |
| [Readiness Checker](agents/readiness-checker.md) | File reading, code search, shell | -- | Pre-implementation readiness gate, requirements traceability |
| [Course Corrector](agents/course-corrector.md) | File reading, code search, shell | -- | Mid-workflow change impact analysis, change proposals |

**Investigator** -- Delegate when you need to understand how something works before changing it. It traces code paths, maps dependencies, and returns structured findings with file paths and line numbers.

**Implementer** -- Delegate when you have a concrete plan and want strict TDD execution. It works in an isolated worktree, runs red-green-refactor cycles, and commits after each change. Hand it a plan, not a vague request.

**Reviewer** -- Delegate after implementation to get a structured code review. It checks correctness, SOLID compliance, OWASP security concerns, performance patterns (N+1 queries), code smells, and test coverage. Returns prioritized findings with severity levels.

**Refactor Scout** -- Delegate for codebase health assessments. It scans directories for bloaters, coupling issues, dispensables, and change preventers, then maps each smell to a named refactoring technique with effort estimates.

**Dependency Auditor** -- Delegate for dependency health checks. It runs `composer audit`, `npm audit`, or equivalent commands, checks for outdated packages, and flags license incompatibilities.

**Doc Writer** -- Delegate after completing features or changes that need documentation. It reads code changes and produces changelogs, API endpoint docs, or migration guides. The only specialist with file editing permissions, but it only touches documentation files.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [krzysztofsurdy/code-virtuoso](https://github.com/krzysztofsurdy/code-virtuoso) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
