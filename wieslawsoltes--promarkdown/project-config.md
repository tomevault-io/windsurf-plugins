---
trigger: always_on
description: This document is mandatory for any human or AI agent contributing to this repository.
---

# Engineering Standards for Agents

This document is mandatory for any human or AI agent contributing to this repository.

These rules are not optional. If a requested implementation conflicts with them, the work must be redesigned to satisfy them rather than bypassing them.

## Mission

Build production-grade C#/.NET software with:

- strict architectural discipline
- professional MVVM composition
- measurable performance awareness
- Fluent-first UI quality
- maintainable, testable, reviewable code

## Non-Negotiable Core Principles

The following principles are mandatory on every change:

- **SOLID**
  - Single Responsibility Principle
  - Open/Closed Principle
  - Liskov Substitution Principle
  - Interface Segregation Principle
  - Dependency Inversion Principle
- **DRY**: do not duplicate logic, parsing rules, UI state rules, or protocol handling.
- **KISS**: prefer the simplest correct design that remains extensible.
- **YAGNI**: do not add speculative abstractions or features without a real need.
- **Separation of Concerns**: keep UI, presentation, domain logic, transport, persistence, and infrastructure clearly separated.
- **Composition over Inheritance**: inherit only when the type relationship is real and beneficial.
- **Fail Fast**: invalid state, broken assumptions, and protocol errors must surface clearly.
- **Explicitness over Magic**: prefer readable contracts and explicit flow over hidden coupling.
- **Correctness before cleverness**: no shortcut is acceptable if it harms reliability or maintainability.

## Architecture Rules for C#/.NET Applications

- Keep classes focused and small enough to have one clear reason to change.
- Depend on abstractions at boundaries. UI must not directly own transport or infrastructure details.
- Prefer constructor injection over service location.
- Model workflows as clear services, state models, and view models with explicit responsibilities.
- Avoid god objects, static mutable state, and hidden global coupling.
- Use immutable data and readonly members where practical.
- Prefer strong typing over stringly typed protocols, flags, or dictionaries.
- Keep public APIs minimal, intentional, and documented by naming and structure.
- Make invalid states difficult to represent.

## Mandatory MVVM Rules

- MVVM is required. Do not place business logic in views.
- Views are for composition, bindings, styling, animation, and view-only interaction concerns.
- ViewModels own presentation state, commands, workflow orchestration, and UI-facing projections.
- Models/services own data contracts, domain behavior, protocol calls, and infrastructure integration.
- Code-behind is allowed only for view-specific concerns such as focus, scrolling, pointer routing, platform view services, and visual-state mechanics that do not belong in the ViewModel.
- Do not access transport, persistence, or environment state directly from XAML views.
- Commands must be explicit, testable, and state-aware.
- Property change flow must be deterministic; avoid hidden side effects.
- Bindings should be strongly typed where available and should avoid reflection-heavy patterns when better options exist.

## C# Coding Standards

- Use nullable reference types correctly. Do not silence warnings without justification.
- Prefer async/await end-to-end for asynchronous workflows.
- Flow `CancellationToken` through cancellable operations.
- Dispose and asynchronously dispose resources correctly.
- Prefer interfaces for boundaries, but do not create empty abstractions with no value.
- Prefer pattern matching, switch expressions, and modern C# features when they improve clarity.
- Avoid broad catch blocks. Catch only what you can handle meaningfully.
- Do not swallow exceptions.
- Prefer clear naming over comments. Add comments only when intent is not obvious from the code itself.
- Do not use `dynamic`, weak typing, or unsafe casts unless there is no better option and the reason is documented in code review.
- Avoid temporal coupling and hidden ordering requirements between method calls.

## Mandatory Performance Standards

Performance is not negotiable. Every agent must actively avoid unnecessary allocations, needless copies, blocking, and UI-thread abuse.

For hot paths, protocol code, parsing, formatting, rendering support, data transforms, and repeated operations, contributors must evaluate and prefer high-performance .NET primitives and patterns, including where appropriate:

- `Span<T>` / `ReadOnlySpan<T>`
- `Memory<T>` / `ReadOnlyMemory<T>`
- `stackalloc`
- `ArrayPool<T>` / `MemoryPool<T>`
- `StringBuilder` or allocation-aware string construction
- `ValueTask` when it is materially beneficial and semantically correct
- `readonly struct`, `record struct`, `in`, `ref`, and `ref readonly` where they reduce copying safely
- `CollectionsMarshal` and similar low-overhead APIs when justified and safe
- SIMD via `Vector<T>` and hardware-accelerated numerics where loops are vectorizable and measurable
- source-generated or low-allocation serialization patterns where applicable

### Performance Rules

- Do not allocate in tight loops unless unavoidable.
- Do not use LINQ on hot paths when it causes avoidable allocations or repeated enumeration.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wieslawsoltes/ProMarkdown](https://github.com/wieslawsoltes/ProMarkdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
