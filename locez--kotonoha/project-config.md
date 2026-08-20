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

## Project Structure

- `src/kotonoha/`: Python package and CLI entry point (`kotonoha.main:entry_point`).
- `src/kotonoha/platform/`: compositor and toolkit decisions — capability probes, the
  native bridge wrapper, and the overlay platform adapters.
- `src/kotonoha/lyrics/`: providers, parsers, matching, and the lyrics cache.
- `src/kotonoha/providers/`: player integrations (MPRIS) and the source gate.
- `src/kotonoha/layer_shell_bridge.cpp`: the C++ Wayland bridge, built to `libkoto-layer.so`.
- `tests/`: Python tests for the root package.
- `plugins/cider/lyrics/`: Cider lyrics probe plugin built with Vite and TypeScript.
- `plugins/cider/lyrics/src/probe/`: Cider playback, TTML parsing, payload, and plugin state logic.
- `plugins/cider/lyrics/src/__tests__/`: Vitest tests for the Cider probe.

Keep generated output out of version control. Python caches, `node_modules/`, Cider
`dist/`, and npm/yarn lockfiles under `plugins/cider/` are ignored.

## Architecture

Business logic, external systems, and presentation code must have clear boundaries.

- Presentation code handles display, input, and state binding.
- Application code coordinates use cases and workflow lifecycles.
- Domain code owns business rules, value objects, state, and stable contracts.
- Infrastructure code adapts networks, files, databases, platform APIs, and third-party libraries.
- Business logic must not depend directly on a concrete UI, network client, or platform implementation.
- Compositor and toolkit facts belong to the platform layer. Presentation code asks
  a capability, and the adapter answers; it must not read the native bridge, a
  desktop-environment name, or a Qt platform name to decide behavior itself.
- A capability that is unavailable must carry a reason the UI can show.
- An operation must report what actually happened. Do not return success because a
  call did not raise when the underlying platform may have ignored it.
- External data must be parsed, validated, and normalized at the boundary.
- Raw third-party objects must not propagate through business code without an explicit contract.
- Prefer interfaces, protocols, or capability models over concrete implementations.
- Do not inherit from a `Protocol` to implement it. Inherited empty bodies turn a
  missing method into a silent no-op; structural typing already checks conformance.
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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [locez/kotonoha](https://github.com/locez/kotonoha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
