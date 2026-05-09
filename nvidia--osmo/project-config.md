---
trigger: always_on
description: This file provides guidance to AI agents when working with the OSMO codebase.
---

# AGENTS.md

This file provides guidance to AI agents when working with the OSMO codebase.

## Overview

OSMO is a workflow orchestration platform for Physical AI, managing heterogeneous Kubernetes clusters for training, simulation, and edge compute workloads.

## Workflow Requirements

Before making any code changes in this repo, you MUST:

1. **Explore first**: Use the Codebase Structure section below to orient yourself, then read relevant source files before proposing changes. Read existing implementations, tests, and related modules. Never modify code you haven't read.
2. **Plan before implementing**: For any non-trivial change (more than a simple one-line fix), create an explicit plan that identifies:
  - Which files need to change and why
  - How the change fits with existing patterns in the codebase
  - What tests exist and what new tests are needed
  - Any cross-cutting concerns (e.g., auth, storage backends, IPC protocols)
  - A verification plan: how to confirm the change works (e.g., specific tests to run, build commands, manual checks)
3. **Check for downstream impact**: This is a multi-service platform — changes in shared libraries (`lib/`, `utils/`) can affect multiple services. Grep for usages before modifying shared code.
4. **Verify after implementation**: After completing changes, execute the verification plan — run the relevant tests/builds and confirm they pass before claiming the work is done. Never assert success without evidence.
5. **Simplify before committing**: Review your changes for unnecessary complexity, redundancy, and over-engineering before committing. Prefer the simplest solution that meets the requirements.
6. **Update documentation**: If adding, removing, or renaming a service, module, or major component, update the "Codebase Structure" section in this file as part of the same change.

## Team Guidelines

- Follow existing code patterns and conventions in the codebase
- Use Bazel for builds and testing
- Go code follows standard Go conventions
- Write self-describing code; avoid redundant comments that simply restate what the code does
- Copyright headers must keep "All rights reserved." on the same line as "NVIDIA CORPORATION & AFFILIATES"
- If copyright lines exceed 100 characters, add `# pylint: disable=line-too-long` comment instead of breaking into multiple lines

## Python Coding Standards

### Import Statements

- All imports must be at the top level of the module
- Place all imports at the top of the file after the module docstring
- **No exceptions**: Imports inside functions are not allowed
  - If circular dependencies exist, the code must be refactored to remove them
  - Common refactoring strategies:
    - Extract shared code into a separate module
    - Use dependency inversion (import abstractions, not concrete implementations)
    - Restructure module hierarchy to break the cycle
    - Use late binding or forward references for type hints (PEP 563)

### Variable Naming

- Do not use abbreviations in variable names unless they are well-understood abbreviations or common conventions
- **Good**: `topology_key`, `config`, `i` (iterator), `x`, `y`, `z` (coordinates)
- **Bad**: `tk` (for topology_key), `topo` (for topology), `req` (for requirement)
- Use full, descriptive names that make code self-documenting

### Type Annotations and Data Structures

- **Use strict typing**: Add type annotations where they improve code clarity and catch errors
- **Prefer dataclasses over dictionaries**: When passing structured data with multiple fields, use dataclasses instead of `Dict[str, Any]`
  - **Good**: `@dataclasses.dataclass class TaskTopology: name: str; requirements: List[...]`
  - **Bad**: `task_data: Dict[str, Any] = {'name': ..., 'requirements': ...}`
- **Avoid unnecessary Optional types**: Only use `Optional[T]` or `T | None` when there is a meaningful behavioral difference between None and an empty value
  - **Good**: `def process(items: List[str])` - caller passes empty list if no items
  - **Bad**: `def process(items: Optional[List[str]])` - now caller must handle None case unnecessarily
  - **When None is meaningful**: Use Optional when None has a distinct meaning from empty (e.g., "not provided" vs "provided but empty")
- **Default arguments for mutable types**: Always use `None` as the default and convert to empty list/dict inside the function
  - **Reason**: Python evaluates default arguments once at function definition time, not per invocation
  - **Good**: `def process(items: List[str] | None = None) -> None: items = items if items is not None else []`
  - **Bad**: `def process(items: List[str] = []) -> None:` - all callers share the same list instance!

### Assertions

- **Do not use `assert` statements in production code** - only in unit tests
- **Reason**: Assertions can be disabled with Python's `-O` flag and should not be relied upon for runtime validation
- **Use proper error handling instead**: Raise appropriate exceptions (ValueError, TypeError, etc.) for validation
  - **Good**: `if value is None: raise ValueError("Value cannot be None")`
  - **Bad**: `assert value is not None, "Value cannot be None"`

## Codebase Structure (`src/`)

All paths below are relative to `src/`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA/OSMO](https://github.com/NVIDIA/OSMO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
