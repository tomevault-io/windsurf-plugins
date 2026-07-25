---
trigger: always_on
description: This file is organized in tiers:
---

# Thinktecture.Runtime.Extensions - AI Assistant Guide

This file is organized in tiers:

- **Tier 1** (this file): Architecture essentials and mandatory policies. Always in context -- no need to load anything else for these.
- **Tier 2** (guides): Task-specific workflows. Load the one guide matching your current task.
- **Tier 3** (reference): Detailed lookup tables. Load only when you need exact specifications.

## Mandatory Pre-Flight Protocol

**Execute these checks BEFORE any codebase exploration or tool calls.** This is not advisory -- these are decision gates.

**Step 1 -- Task routing.** Classify the user's intent against the Tier 2 routing table below. Load the matched guide. You MUST NOT make exploratory tool calls (find_symbol, search_for_pattern, get_symbols_overview) until the guide is loaded. Codebase exploration before guide loading is a protocol violation.

**Step 2 -- Delegation check.** If the user's request involves 3 or more of: design, implementation, testing, review, documentation -- you MUST delegate to subagents via the Task tool. Do not produce an inline plan. Creating a comprehensive inline plan is NOT delegation.

> **WRONG**: User says "I want to add feature X." Agent immediately calls find_symbol to explore.
> **RIGHT**: Agent loads `guides/DESIGN-DISCUSSION.md` first, then explores the codebase following the guide's workflow.

> **WRONG**: User asks for feature + tests + docs. Agent produces a 200-line inline plan.
> **RIGHT**: Agent says "This is a multi-phase task requiring subagents" and spawns the first agent.

---

## Common Commands

- **Build**: `dotnet build`
- **Restore**: `dotnet restore`
- **Test**: `dotnet test`
- **Test (filtered)**: `dotnet test --filter "FullyQualifiedName~MyTestClass"`

### Development Requirements

- .NET 10.0 SDK (as specified in global.json)
- C# 14.0 (as specified in Directory.Build.props)
- Multiple .NET versions (8.0, 9.0, 10.0) for framework compatibility testing

---

## Tier 1: Architecture Essentials

A .NET library providing Smart Enums, Value Objects, and Discriminated Unions via Roslyn Source Generators.

### Type Categories

| Type                 | Attribute                                     | Description                                                                                                      |
|----------------------|-----------------------------------------------|------------------------------------------------------------------------------------------------------------------|
| Keyed Smart Enum     | `[SmartEnum<TKey>]`                           | Type-safe enum with underlying key value (int, string, Guid, custom). Items are `public static readonly` fields. |
| Keyless Smart Enum   | `[SmartEnum]`                                 | Type-safe enum without underlying value. Identified by field reference only.                                     |
| Simple Value Object  | `[ValueObject<TKey>]`                         | Single-value immutable type wrapping one underlying value.                                                       |
| Complex Value Object | `[ComplexValueObject]`                        | Multi-property immutable type.                                                                                   |
| Ad-hoc Union         | `[Union<T1, T2, ...>]` or `[AdHocUnion(...)]` | "One of" several types. Up to 5 type parameters. Cannot be generic.                                              |
| Regular Union        | `[Union]`                                     | Inheritance-based union. Abstract base with sealed derived types.                                                |

All types must be declared as `partial`. Source generators produce: factory methods (`Create`, `TryCreate`, `Validate`), equality members, conversion operators, `Switch`/`Map` pattern matching, `IParsable<T>`/`ISpanParsable<T>` (NET9+), and serialization integration.

### Type Details

**Keyed Smart Enums** (`[SmartEnum<TKey>]`):

- Items are `public static readonly` fields. No on-demand creation.
- Interface implementations depend on key type capabilities: `IParsable<T>`, `ISpanParsable<T>` (NET9+), `IComparable<T>`, `IFormattable`
- Span-based JSON deserialization (NET9+, string keys): automatic, opt out via `DisableSpanBasedJsonConversion = true`

**Keyless Smart Enums** (`[SmartEnum]`):

- No key, no lookup by value. Identified solely by field reference.

**Simple Value Objects** (`[ValueObject<TKey>]`):

- String keys MUST specify `[KeyMemberEqualityComparer<...>]`
- Validation: prefer `ValidateFactoryArguments` over `ValidateConstructorArguments`
- Zero-allocation JSON (NET9+): opt-in via `[ObjectFactory<ReadOnlySpan<char>>(UseForSerialization = SerializationFrameworks.SystemTextJson)]`

**Complex Value Objects** (`[ComplexValueObject]`):

- Use `[IgnoreMember]` to exclude properties. Use `[MemberEqualityComparer<...>]` for custom equality.

**Ad-hoc Unions** (`[Union<T1, T2>]` or `[AdHocUnion(...)]`):

- Stateless types (`TXIsStateless = true`): store only discriminator, not instance. Prefer structs.

**Regular Unions** (`[Union]`):

- Base type abstract, derived types sealed. Conversion operators + Switch/Map.

### What Gets Generated

All types: equality members (`Equals`, `GetHashCode`, `==`, `!=`), `Switch`/`Map` pattern matching.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PawelGerr/Thinktecture.Runtime.Extensions](https://github.com/PawelGerr/Thinktecture.Runtime.Extensions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
