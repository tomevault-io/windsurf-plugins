---
trigger: always_on
description: Before running Maven, E2E, Proxy startup, database clients, IDE/MCP run configurations, any command likely to output more than 100 lines,
---

# ShardingSphere AI Development Guide

## TOKEN EFFICIENCY ACTION — READ OR REUSE .codex/context/token-efficiency.md

Before running Maven, E2E, Proxy startup, database clients, IDE/MCP run configurations, any command likely to output more than 100 lines,
or any task likely to produce large analysis or review output, I MUST ensure `.codex/context/token-efficiency.md` is available in the active context.
If this exact file from this repository has already been read in the current session and there is no evidence it changed, reuse the loaded content.
Otherwise, read it before running the high-output command or producing the large structured output.
Execute the command according to the Mandatory Execution Contract in that file.
This file is the repository-local source of truth for token-efficient command classification, log capture, filtered summaries, final reporting, and structured output.
Paths in this section are relative to the Apache ShardingSphere repository root.

This guide is written **for AI coding agents only**. Follow it literally; improvise only when the rules explicitly authorize it.

## Core Immutable Principles

1. **Quality First**: code quality and system security are non-negotiable.
2. **Think Before Action**: perform deep analysis and planning before coding.
3. **Tools First**: prioritize the proven best toolchain.
4. **Transparent Records**: keep every key decision and change traceable.
5. **Continuous Improvement**: learn from each execution and keep optimizing.
6. **Results Oriented**: judge success solely by whether the target is achieved.
7. **Coding Standards**: `CODE_OF_CONDUCT.md` is the binding “law”; use it as the first reference for rule interpretation and cite relevant lines. See Governance Basics for precedence and session review expectations.

## Quality Standards

### Engineering Principles
- **Architecture**: follow SOLID, DRY, separation of concerns, and YAGNI (build only what you need).
- **Code Quality**:
    - Use clear naming and reasonable abstractions.
    - **Meaningful Code Gate**:
        - **Purpose traceability**: every added line, identifier, literal, YAML anchor, helper, abstraction, and configuration entry must have a traceable purpose.
          Before keeping AI-generated code, trace it back to one concrete need: production behavior, public contract, regression protection, user-facing diagnostics,
          security or safety, readability, or removal of real duplication.
          If the purpose cannot be explained in one specific sentence, remove it.
        - **Meaningless code definition**: meaningless code is code that does not change or protect behavior, clarify a real contract,
          reduce real duplication or complexity, improve diagnostics or safety, or make the code easier to read.
          Code is also meaningless when it exists only for formal symmetry, complete-looking groups, tidy-looking structure, coverage appearance,
          or possible future flexibility.
          Do not keep code merely because it was generated, looks tidy, might be useful later, or makes a diff look more complete.
        - **YAML anchor rule**: do not add YAML anchors unless they are actually reused in the same file and reduce meaningful duplication.
          A YAML anchor with no aliases, an anticipatory anchor for possible future reuse, or an anchor that makes nearby YAML harder to read is forbidden.
          Prefer repeating a small YAML block when repetition is clearer than indirection.
    - Do not introduce package-private top-level helper types by default.
      Keep very small, single-owner state or continuation helpers as private nested types, but avoid accumulating multiple nested collaborators inside one class.
      When a helper has cohesive behavior, multiple callers, direct test value, or enough logic to distract from the owner class, split it into a public top-level type with a clear contract and direct tests.
      If neither private nor public fits, do not add a helper; keep the implementation in the approved owner or simplify the design within the declared boundary.
    - Every new public production type must have direct, focused tests.
      Broad workflow tests do not replace public contract tests unless they explicitly exercise that public type's behavior.
    - Do not change production code solely for test convenience.
      Test-only reuse, easier mocking, coverage convenience, fixture sharing, or test-only construction must not justify adding production types,
      widening visibility, changing constructors, adding overloads, altering signatures, moving test helpers into production, or introducing abstractions.
      Production changes must have an independent production reason, such as fixing behavior, clarifying a real contract, reducing production duplication,
      or exposing a construction path that production code legitimately supports.
      Prefer test-local fixtures, mocks, existing public constructors, factories, builders, SPI loaders, or production APIs when construction is incidental to the behavior under test.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/shardingsphere](https://github.com/apache/shardingsphere) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
