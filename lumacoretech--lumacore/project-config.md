---
trigger: always_on
description: 1. [General](#general)
---

# Copilot Instructions

## Table of Contents

1. [General](#general)
   - [Repository Constraints](#repository-constraints)
   - [Third-Party Notices](#third-party-notices)
   - [Build Configuration](#build-configuration)
   - [Terminal Usage](#terminal-usage)
2. [Working with Code](#working-with-code)
   - [Code Style & Changes](#code-style--changes)
   - [XML Documentation](#xml-documentation)
   - [Core Principles](#core-principles)
   - [Code Patterns](#code-patterns)
   - [Verification & Review](#verification--review)
3. [Workflow](#workflow)
4. [Testing](#testing)
   - [Authoring Rules](#authoring-rules)
   - [Test Structure](#test-structure)
   - [Common Test Topics](#common-test-topics)
   - [Coverage](#coverage)
5. [Data Layer / EF Core](#data-layer--ef-core)
   - [Entity Documentation](#entity-documentation)
   - [Entity Layout](#entity-layout)
   - [DbContext Configuration](#dbcontext-configuration)
   - [Migrations](#migrations)
   - [Data Layer Guidelines](#data-layer-guidelines)
   - [Service API Conventions](#service-api-conventions)
6. [Privacy & Data Minimization](#privacy--data-minimization)
7. [Logging & Configuration](#logging--configuration)
8. [Code Layout & Naming](#code-layout--naming)
9. [Commit Messages](#commit-messages)
10. [Member Ordering](#member-ordering)

---

## General

### Repository Constraints

> [!IMPORTANT]
> **Target Framework:** .NET 10

- Do **not** perform any **write** operations using Git (e.g., commit, push, reset, rebase, merge, tag, branch creation/deletion).
- Localization is only active in the UI (Blazor) — API responses and validation messages remain in **English**.

### Third-Party Notices

When **adding, removing, or updating** a third-party dependency (NuGet package, npm package, bundled JS library, or any other external asset), **always update `THIRD-PARTY-NOTICES.md`** in the repository root accordingly:

- **Adding:** Insert a new entry with Author, License, and URL.
- **Removing:** Delete the corresponding entry.
- **Updating:** Adjust the entry if the license or author changed (version bumps with no license change need no update).

### Terminal Usage

> [!WARNING]
> **Avoid complex PowerShell scripts** in the Copilot terminal — they tend to hang and require manual cancellation.

- Prefer using `replace_string_in_file` or `multi_replace_string_in_file` for file modifications.
- Use simple, single-command terminal calls only when necessary (e.g., `dotnet build`, `Get-Content`).
- Do **not** use loops, pipelines with multiple stages, or scripts that modify many files at once via the terminal.

### Build Configuration

- **Avoid behavior differences** between Debug and Release builds.
- **For unreachable guards** (code that should never execute because all cases are covered):
  - Use `throw new UnreachableException()` from `System.Diagnostics` instead of `#if DEBUG` / `#if RELEASE` blocks.
  - Always add an explicit comment explaining **why** the path is unreachable (e.g., "All enum values handled above").

---

## Working with Code

> [!IMPORTANT]
> These principles apply to **all** code interactions — writing, modifying, reviewing, or debugging.

### Code Style & Changes

- **Keep changes minimal** and focused on the task at hand.
- **Match the existing code style** of the surrounding code.
- **Line length limit: 120 characters.** Use the available width — don't break lines unnecessarily short.
- **Comment non-obvious decisions.** When a choice isn't immediately self-evident (empty templates, suppressed defaults, workarounds, defensive guards, intentional no-ops), add a comment explaining *why* — not *what*. If someone might later ask "why was this done this way?", a comment is mandatory.
- When you notice opportunities for improvement (refactoring, better patterns, code quality):
  - **Explicitly mention them** and start a dialog.
  - After user approval, improvements can be implemented.

> [!NOTE]
> When a task is explicitly about repository instructions (like this file), it is OK to update instruction files immediately without waiting for an explicit start signal.

### XML Documentation

- Add complete XML documentation (`<summary>`, `<param>`, `<returns>`, `<exception>`) for **all** methods and properties you add or change, including private and internal members.
- Use `<remarks>` for additional details, `<example>` for usage samples when helpful.
- XMLDocs should be written for **all members**, not just public APIs — internal and private members are important for developers.
- **Use the full 120-character line width** in XML documentation. Break at logical points (end of sentence) rather than at arbitrary positions.
- For `cancellationToken` parameters, use the standard phrasing: `A token to cancel the operation.` Context-specific phrasings are allowed as exceptions.

#### Common Inline Tags

| Tag | Usage |
|-----|-------|
| `<see cref="..."/>` | Link to types, methods, properties |
| `<see langword="..."/>` | Language keywords (`null`, `true`, `false`, etc.) |
| `<c>...</c>` | Inline code |
| `<code>...</code>` | Multi-line code block |
| `<paramref name="..."/>` | Reference a parameter |
| `<typeparamref name="..."/>` | Reference a type parameter |

#### Style Guidelines

- Prefer the **shortest possible** type names (use `using` directives to enable short names).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LumaCoreTech/LumaCore](https://github.com/LumaCoreTech/LumaCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
