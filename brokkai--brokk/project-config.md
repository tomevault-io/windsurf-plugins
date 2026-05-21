---
trigger: always_on
description: This repository contains multiple subprojects with different languages and standards.
---

# Brokk Coding Guide

## Multi-Project Routing

This repository contains multiple subprojects with different languages and standards.

- **Python (Terminal UI)**: If you are editing files under `brokk-code/`, follow [brokk-code/AGENTS.md](brokk-code/AGENTS.md) for Python-specific rules (Textual, Ruff, Pytest).
- **Java (Executor/Core)**: For Java code, follow this root guide and any nested `AGENTS.md` files within Java packages. Analyzer code lives under `brokk-shared/src/main/java/ai/brokk/analyzer`, not `app/`.

## Null Safety

1. **Null Away**: This project is built with Null Away: fields, parameter, and return values are non-null by default. 
Annotate exceptions to this rule with @Nullable (imported from org.jetbrains.annotations). Use requireNonNull 
(static import from java.util.Objects) when the static type is @Nullable but our code path expects it to be non-null.
Less often, it is useful to use castNonNull (static import from org.checkerframework.checker.nullness.util.NullnessUtil) 
when we can prove a value is not null but the compiler doesn't realize it, e.g. accessing get(true) in a Map 
returned by Collectors.partitioningBy. You do not need either requireNonNull or castNonNull when a field, parameter,
or return value is not annotated @Nullable.
1. **RedundantNullCheck**: Try to resolve `RedundantNullCheck` warnings by either removing the redundant null check or by annotating the reported variable or method with @Nullable (imported from org.jetbrains.annotations). Do NOT suppress the `RedundantNullCheck` warnings.
1. **@NullMarked**: Add `@org.jspecify.annotations.NullMarked` to `package-info.java` for any new Java source code packages.
1. **Optional**: Prefer returning Optional from methods to @Nullable; the opposite is generally the case for method parameters.

## General Principles

1. **Java 21 features**: The codebase leverages Java features up to JDK 21. Embrace the lambdas! and also getFirst/getLast, Collectors.toList, pattern matching for instanceof, records and record patterns, etc.
1. **Prefer functional streams to manual loops**: Leverage streams for transforming collections, joining to Strings, etc.
1. **Favor Immutable Data Structures**: Prefer `List.of` and `Map.of`, as well as the Stream Collectors.
1. **Provide Comprehensive Logging**: Log relevant information using log4j, including request/response details, errors, and other important events. Prefer logging full collections (trust the toString), not just counts.
1. **@Blocking and EDT safety**: Annotate public methods that may block (I/O, analyzer work, network, filesystem, or other expensive computation) with `org.jetbrains.annotations.Blocking`. On the Swing Event Dispatch Thread (EDT), do not invoke `@Blocking` methods; prefer the non-blocking `computed*` alternatives (e.g., `computedFiles()`, `computedSources()`, `computedText()`, `computedDescription()`, `computedSyntaxStyle()`) to keep the UI responsive. An Error Prone check (`BrokkBlockingOperation`) enforces this and will warn if an `@Blocking` method is called on the EDT (e.g., inside `SwingUtilities.invokeLater(...)` or the true branch of an `isEventDispatchThread()`/`isDispatchThread()` check). Fix by moving the call off the EDT or by using the appropriate `computed*` method; do not suppress the warning.
1. **Use asserts to validate assumptions**: Use `assert` to validate assumptions, and prefer making reasonable assumptions backed by assert to defensive `if` checks.
1. **DRY**: Don't Repeat Yourself. Refactor similar code into a common method. But feature flag parameters are a design smell; if you would need to add flags, write separate methods instead.
1. **Parsimony**: If you can write a general case that also generates correct results in the special case (empty input, maximum size, etc), then do so. Don't write special cases unless they are necessary.
1. **Use imports**: Avoid raw, fully qualified class names unless necessary to disambiguate; otherwise import them. EXCEPTION: if you are editing from individual method sources or usages call sites, use FQ names since you can't add easily add imports.
1. **YAGNI**: Follow the principle of You Ain't Gonna Need It; implement the simplest solution that meets the requirements, unless you have specific knowledge that a more robust solution is needed to meet near-future requirements.
1. **Keep related code together**: Don't split out code into a separate function, class, or file unless it is completely self-contained and either signifcantly complex or called from multiple sites. It's easier to understand a short computation inline with its context, than split out to a separate location.
1. **Prefer unordered sets**: use HashSet and Collectors.toSet unless we specifically need ordering.
1. Prefer declaring utility records inline with the code that creates or returns them rather than as tiny standalone files.

## Working with LLMs

1. **Tool Calls**: If you're making tool calls, make sure you have summaries of ToolRegistry and ToolExecutionResult in the Workspace.
Note that ToolRegistry::executeTool catches exceptions and turns them into TER with INTERNAL_ERROR status, so wrapping executeTool
with try/catch is unnecessary and futile; don't do that.

## Things to avoid


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BrokkAi/brokk](https://github.com/BrokkAi/brokk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
