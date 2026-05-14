---
trigger: always_on
description: helps with the judgment layer). The pipeline validates against
---

# Claude Rules

> **This file**: Core principles, behavioral constraints, and development rules.
> The principles section is the foundation — internalize it before any work.

## Core Principles

These principles govern every change to this project. They are not
guidelines — they are hard constraints. When in doubt, the principle
wins over convenience.

### Architecture

1. **Separation of Concerns is non-negotiable.** Each module does one
   thing. Transport-specific logic stays in transport-scoped modules.
   Shared logic lives in shared modules. No generic abstractions that
   span transport boundaries — if two things share a name but not
   logic, they belong in separate modules.

2. **DRY is non-negotiable.** If the same logic appears in 2+ places,
   extract a shared helper. Duplicated protocol constants, signing
   functions, or dispatch logic are architecture bugs, not tech debt.

3. **Core is platform-agnostic.** No `homeassistant.*` imports in
   Core. HA-specific code stays in `custom_components/`. Core's API
   is synchronous (`requests`-based I/O). The HA adapter is a thin
   wrapper — if HA wiring requires non-trivial logic, Core's API
   boundaries are wrong.

4. **New features are additive only.** New format, new auth strategy,
   new transport, new parser — none of these change existing code.
   Add a new implementation, register it, done. If adding a feature
   requires modifying unrelated modules, the architecture is wrong.

5. **Protocol primitives are shared, implementation is scoped.**
   Constants and signing functions live in shared protocol modules
   (`protocol/hnap.py`). How a transport uses them is transport-
   specific (auth, loaders, and action executors each own their flow).

### Modem Configuration

6. **Modem behavior is data-driven.** Core provides a set of
   behaviours (auth strategies, extraction modes, parsers, executors).
   Each modem selects from them via YAML. No modem's configuration
   requires code changes. No modem's configuration affects another.

7. **Config fields are parameters to core behaviours, not raw
   implementations.** `auth.strategy: form` selects an auth strategy.
   `endpoint_pattern: "RouterStatus"` supplies a keyword to a core
   extraction strategy. Contributors provide *what*, core handles
   *how*. If a config field requires regex, code, or implementation
   knowledge, the abstraction is wrong.

8. **Catalog Tools intake is the onboarding path.** New modems come
   through the Catalog Tools pipeline (`/modem-intake` skill or the
   equivalent function calls), not by hand-constructing files.
   Catalog Tools lives in `packages/cable_modem_monitor_catalog_tools/`
   — never installed by HA, but open to contributors with hardware
   and AI assistance (the pipeline mechanics are plain Python; AI
   helps with the judgment layer). The pipeline validates against
   specs end-to-end. Manual construction bypasses that validation.

### Specs and Documentation

9. **Specs are the authority.** Code follows specs. No silent
   deviations. If the code needs to diverge, discuss the gap first,
   update the spec, then update the code.

10. **Design decisions land in specs, not in conversation.** Every
    architectural decision made during a session must be committed to
    the relevant spec file before the session ends. Conversation
    history is ephemeral — specs are durable.

11. **Docs and code move together.** Every core change reconciles the
    affected specs (ARCHITECTURE, ORCHESTRATION_SPEC, MODEM_YAML_SPEC,
    etc.). A code change without a corresponding spec update is
    incomplete.

### Code Quality

12. **No shortcuts, no deferred structure.** If a better design is
    obvious (split by transport, shared types module, DRY utility),
    use it now. Don't optimise for speed of first draft. When a module
    grows past its natural boundary, restructure the whole module —
    don't bolt on the new thing and leave the rest.

13. **Quality gates are not negotiable.** If mypy, ruff, black, or
    pytest fails, fix the code. Don't exclude files, skip checks,
    or weaken thresholds. The only valid exclusions are generated code
    and vendored dependencies. This applies to all linters including
    markdownlint — fix the source files, don't silence rules that
    flag real issues. Only configure away rules that are genuinely
    inapplicable (e.g. line length for URLs, duplicate headings in
    changelogs).

14. **Test overrides are a code smell.** If reaching coverage requires
    heavy mocking, monkeypatching, or test overrides, the code
    structure is wrong. Restructure the code (extract dependency, make
    injectable), don't paper over it with test complexity.

15. **No forward references.** Helper functions that reference a class
    must be defined after the class, not before it.
    `from __future__ import annotations` makes it parse, but the code
    reads wrong.

### Testing

16. **Table-driven tests by default.** Identify the pattern BEFORE
    writing tests, not during review. If 3+ tests share the same
    setup→call→assert structure, start with the table.

17. **Schema tests use fixtures, behavioural tests stay inline.**
    Valid/invalid configs are JSON fixture files (add a file to add a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [solentlabs/cable_modem_monitor](https://github.com/solentlabs/cable_modem_monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
