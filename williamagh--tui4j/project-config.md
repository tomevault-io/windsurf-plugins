---
trigger: always_on
description: TUI4J - Java port of Bubble Tea (Charm) for Terminal UIs
---


# TUI4J Agent Rules

## ⚠️ PUBLIC API — NO BREAKING CHANGES

This is a **published Maven Central library** with downstream consumers (Brief, others).
- 100% backward compatibility required for all refactors.
- Never remove or rename public classes, methods, or fields.
- Never change method signatures, return types, or exception types.
- Run `./gradlew build` and confirm all tests pass before any PR.
- When in doubt, do not change the public API.

## Document Organization [ORG]

- [ORG1] Purpose: keep every critical rule within the first ~250 lines; move long examples/notes to Appendix.
- [ORG2] Structure: Rule Summary first, then detailed sections keyed by short hashes (e.g., `[GT1a]`).
- [ORG3] Usage: cite hashes when giving guidance or checking compliance; add new rules without renumbering older ones.

## Rule Summary [SUM]

- [ZA1a-d] Zero Tolerance Policy (zero assumptions, validation, forbidden practices, dependency verification)
- [GT1a-h] Git & Permissions (elevated-only git; no destructive commands)
- [CC1a-d] Clean Code & DDD (Mandatory)
- [ID1a-d] Idiomatic Patterns & Defaults
- [RC1a-e] Root Cause Resolution (no fallbacks, no shims/workarounds)
- [FS1a-g] File Creation & Clean Architecture (search first, strict types, single responsibility)
- [LOC1a-e] Line Count Ceiling (350 lines max; SRP enforcer; zero tolerance)
- [MO1a-g] No Monoliths (Strict SRP; Decision Logic; Extension/OCP)
- [AB1a-d] Abstraction Discipline (reuse-first, no anemic wrappers)
- [JD1a-d] Javadoc Standards (mandatory, why > what, deprecation policy)
- [TS1a-d] Testing Standards (coverage mandatory, observable behavior)
- [DS1a-e] Dependency Source Verification (locate, list, read, search)

## [ZA1] Zero Tolerance Policy

- [ZA1a] **Zero Assumptions**: Do not assume behavior, APIs, or versions. Verify in the codebase/docs first.
- [ZA1b] **Source Verification**: For dependency code questions, inspect `~/.m2` JARs or `~/.gradle/caches/` first; fallback to upstream GitHub; never answer without referencing code.
- [ZA1c] **Forbidden Practices**:
  - No `Map<String, Object>`, raw types, unchecked casts, `@SuppressWarnings`, or `eslint-disable` in production.
  - No trusting memory—verify every import/API/config against current docs.
- [ZA1d] **Mandatory Research**: You MUST research dependency questions and correct usage. Never use legacy or `@deprecated` usage from dependencies. Ensure correct usage by reviewing related code directly in `node_modules` or Gradle caches and using online tool calls.
- [ZA1e] No empty confirmations ("You're right", "Absolutely") before investigation; verify then cite evidence

## [GT1] Git & Permissions

- [GT1a] All git commands require elevated permissions; never run without escalation.
- [GT1b] Never remove `.git/index.lock` automatically—stop and ask the user or seek explicit approval.
- [GT1c] Read-only git commands (e.g., `git status`, `git diff`, `git log`, `git show`) never require permission. Any git command that writes to the working tree, index, or history requires explicit permission.
- [GT1d] Do not skip commit signing or hooks; no `--no-verify`. No `Co-authored-by` or AI attribution.
- [GT1e] Destructive git commands are prohibited unless explicitly ordered by the user (e.g., `git restore`, `git reset`, force checkout).
- [GT1f] Treat existing staged/unstaged changes as intentional unless the user says otherwise; never “clean up” someone else’s work unprompted.
- [GT1g] Examples of write operations that require permission: `git add`, `git commit`, `git checkout`, `git merge`, `git rebase`, `git reset`, `git restore`, `git clean`, `git cherry-pick`.
- [GT1h] **Repository-Local Writes Only**: NEVER commit or push to this repository from a temporary clone, alternate checkout/worktree, or any other directory copy of the same repo. All git writes must be executed from this exact working tree.

## [CC1] Clean Code & DDD (Mandatory)

- [CC1a] **Mandatory Principles**: Clean Code principles (Robert C. Martin) and Domain-Driven Design (DDD) are **mandatory** and required in this repository.
- [CC1b] **DRY (Don't Repeat Yourself)**: Avoid redundant code. Reuse code where appropriate and consistent with clean code principles.
- [CC1c] **YAGNI (You Aren't Gonna Need It)**: Do not build features or abstractions "just in case". Implement only what is required for the current task.
- [CC1d] **Clean Architecture**: Dependencies point inward. Domain logic has zero framework imports.
- [CC1e] **Tracer bullet**: Build one tiny end-to-end slice through all layers first; validate it works; then expand — never build horizontal layers in isolation.
- [CC1f] **KISS**: Simplest solution that works; achieve by removing, not adding; use platform/framework defaults unless deviation is proven necessary

## [SS1] Single Semantic Owner (Mandatory)

- [SS1a] **One Owner**: For any governed concept, exactly one file/module may define its field inventory, names, allowed keys, dependency graph, or behavior selection.
- [SS1b] **No Mirrors**: Tests, fixtures, docs, examples, generators, compat wrappers, and helper modules are NOT exempt; they must bind/import/project the canonical owner instead of restating it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WilliamAGH/tui4j](https://github.com/WilliamAGH/tui4j) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
