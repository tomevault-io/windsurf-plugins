---
trigger: always_on
description: This file applies to the repository root and all subdirectories.
---

# AGENTS.md

## Scope

This file applies to the repository root and all subdirectories.

- This file contains only long-lived, repeatable engineering rules.
- Do not store issue lists, milestones, temporary migrations, or current technical-debt inventories here.
- Specific work is tracked in issues, pull requests, and design documents.
- A nested `AGENTS.md` may add stricter local rules, but must not weaken this file.
- System, developer, and user instructions take precedence over this file.

## General Principles

- Understand the existing code, tests, build configuration, and runtime before changing them.
- Prefer small, verifiable, reversible changes.
- Never overwrite, revert, or clean up changes that belong to the user.
- Avoid unrelated refactors, formatting changes, and dependency upgrades.
- Do not turn a temporary workaround into a permanent design.
- Add an abstraction only when it reduces coupling, clarifies ownership, or improves testability.
- Update tests and documentation when behavior or public contracts change.

## Change Workflow

For every task:

1. Confirm the scope, constraints, and acceptance criteria.
2. Check the current branch and working-tree status.
3. Read the relevant implementation, tests, and configuration.
4. Define the behavior boundary before implementing it.
5. Decide what evidence is needed for the change. Add or update tests only when
   they verify a meaningful behavior, contract, invariant, or failure path.
   Do not add tests merely because a file, setting, or code path changed.
6. Keep the change focused and complete.
7. Run the checks relevant to the change.
8. Review the final diff for unrelated files and temporary artifacts.
9. Record the change, tests, risks, and remaining work in the commit or pull request.

Do not develop directly on the default branch.

## Architecture

Business logic, external systems, and presentation code must have clear boundaries.

- Presentation code handles display, input, and state binding.
- Application code coordinates use cases and workflow lifecycles.
- Domain code owns business rules, value objects, state, and stable contracts.
- Infrastructure code adapts networks, files, databases, platform APIs, and third-party libraries.
- Business logic must not depend directly on a concrete UI, network client, or platform implementation.
- External data must be parsed, validated, and normalized at the boundary.
- Raw third-party objects must not propagate through business code without an explicit contract.
- Prefer interfaces, protocols, or capability models over concrete implementations.
- Avoid circular dependencies, implicit global state, and owner objects that absorb unrelated responsibilities.
- Transitional compatibility code must have a clearly limited scope and a documented removal condition.

## Strong Typing

Strong typing is a continuous requirement for all new and modified code.

- Public functions, methods, attributes, and cross-module interfaces must have complete annotations.
- Do not introduce unexplained `Any` into application or domain contracts.
- Do not use unbounded dictionaries to represent business state.
- Parse external JSON, configuration, and third-party objects into typed structures.
- Use `dataclass`, `Enum`, `Protocol`, `TypedDict`, and type aliases when they express the real contract.
- Give states, commands, events, errors, and configuration explicit structures.
- Before adding a helper or utility function, search the existing code for an equivalent capability. Reuse the existing function or extend its typed contract when it owns the same responsibility; add a new helper only when the responsibility or ownership is genuinely different.
- When a value does not satisfy a function's annotated input type, first decide whether the function's contract should accept that value. Prefer widening or clarifying the function's type and testing the resulting contract when the value is semantically valid; do not add call-site conversions merely to satisfy the annotation. Convert at a real boundary only when normalization is part of that boundary's explicit contract, with validation that preserves the intended meaning.
- Use the type checker configured by the repository; do not hide errors by expanding exclusions.
- Every type suppression must document its reason, impact, and follow-up path.

## Explicit Contracts and Dynamic Access

- Initialize every instance attribute in `__init__` or an explicit factory, with a declared type. Optional state must be initialized to `None`; callers should use direct attribute access and explicit `is None` checks.
- Do not use reflection-style access (`getattr`, `hasattr`, `setattr`, `delattr`, `__dict__`, `globals()`, or `locals()`) to model application or domain state, discover fields, or hide incomplete initialization. Do not add trivial getter/setter methods or generic `get(name)` APIs for ordinary fields; use typed attributes or properties. Use a method when it performs computation, validation, a meaningful side effect, or protects an invariant.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [locez/bilihud](https://github.com/locez/bilihud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
