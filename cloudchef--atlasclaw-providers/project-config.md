---
trigger: always_on
description: This repository owns concrete integrations. If a document or implementation
---

# AGENTS.md - AtlasClaw Providers Guidelines

## Repository Overview

This repository owns concrete integrations. If a document or implementation
depends on a target system's auth model, fields, workflow semantics, webhook
payloads, or provider-specific UI text, it belongs here rather than in
`atlasclaw` core.

This repository is organized into a few top-level modules:

- `providers/`: concrete provider packages such as `SmartCMP-Provider` and `jira`
- `skills/`: shared reusable skills, templates, and helper tooling that are not tied to a single provider
- `docs/`: repository-level architecture notes, design plans, and supporting assets
- `.agentdocs/`: optional AI-only working notes and memory files used during execution

Most provider work happens under a provider package with a layout similar to:

```text
providers/<provider-name>/
|-- PROVIDER.md
|-- README.md
|-- skills/
|   `-- <skill-name>/
|       |-- SKILL.md
|       |-- scripts/
|       |-- references/
|       `-- test/              # optional, skill-scoped tests
|-- docs/                      # optional, provider-specific plans or design notes
`-- test/                      # optional, provider-level tests
```

Directory responsibilities:

- `PROVIDER.md`: provider contract, configuration shape, and authentication model
- `README.md`: human-readable provider overview
- `skills/<skill-name>/SKILL.md`: skill metadata, usage guidance, and tool entrypoints
- `skills/<skill-name>/scripts/`: executable integration logic and helpers
- `skills/<skill-name>/references/`: API mappings, workflows, and examples kept close to the skill
- `docs/` and `test/`: provider-specific design notes and verification coverage

Core/runtime contracts, loading behavior, and provider-agnostic rules stay in
`atlasclaw`. Concrete provider behavior and examples stay here.

## Code Style Guidelines

- Comments must be in English
- Prefer wide lines suitable for large screens; avoid over-wrapping
- Do not split simple logic into many tiny methods unless reuse justifies it
- Avoid breaking short code across many lines
- Keep validation pragmatic: validate external input, persistence integrity, and core trading invariants, but do not add layers of redundant defensive checks that do not improve correctness

### Python

**Imports:**
- Prefer `from __future__ import annotations` in new or heavily edited modules when it improves type annotations
- Standard library imports first, third-party second, local third
- Group imports with a blank line between groups
- Prefer absolute imports for AtlasClaw core packages, but keep same-directory helper imports such as `from _common import ...` when that matches the existing provider script layout

**Formatting:**
- Keep or add a UTF-8 encoding header when the file already uses it or the file contains non-ASCII content that warrants being explicit
- 4 spaces for indentation
- Line length: ~100 characters (be reasonable)
- Use double quotes for strings unless single quotes avoid escaping

**Types:**
- Use type hints on public functions and non-trivial helpers
- Prefer consistency with the surrounding file for `Optional[T]` versus `T | None`; do not churn existing code only to change annotation style
- Use dataclasses for data containers: `@dataclass`
- Prefer enums for string constants: `class EventType(str, Enum)`

**Naming Conventions:**
- `snake_case` for functions, variables, modules
- `PascalCase` for classes, exceptions
- `SCREAMING_SNAKE_CASE` for constants
- Private methods/attributes prefixed with `_`

**Error Handling:**
- Use specific exception types, not bare `except:`
- Include error context in exception messages
- Return result objects for expected failures: `SendResult(success=False, error="timeout")`
- Use `asyncio.Event` for cancellation signals

**Documentation:**
- Docstrings use triple quotes on separate lines
- Include docstrings for all public classes and methods
- Use Google-style or reStructuredText format

**Async Patterns:**
- Use async for AtlasClaw runtime entrypoints and other integration points that are already async
- Standalone helper scripts may stay synchronous when the surrounding implementation and dependencies are synchronous
- Use `asyncio.Event` for coordination when cancellation or cross-task signaling is needed
- Properly await coroutines in tests with `@pytest.mark.asyncio`

## Code Documentation Rules

- Every public class must have a class-level docstring explaining its responsibility
- Every public method or function must have a docstring describing:
  - purpose
  - parameters
  - return value
  - possible exceptions or error cases
- Add short comments for non-obvious business rules, edge cases, and tricky logic
- Do not add trivial comments that merely restate the code
- Prefer self-explanatory naming first; comments are for intent, constraints, and reasoning

## Analyze Field Impact Before Changing Existing Fields

Before changing an existing field, including its format, storage style, or calculation logic, you must:

1. analyze all call sites, including `getXxx()` and `setXxx()` references
2. confirm the stored data format contract
3. check formulas or strategy logic that depend on that format
4. explain the impact to the user before deciding the final change


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CloudChef/atlasclaw-providers](https://github.com/CloudChef/atlasclaw-providers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
