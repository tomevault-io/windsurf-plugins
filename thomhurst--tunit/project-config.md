---
trigger: always_on
description: - [Critical Rules - READ FIRST](#critical-rules---read-first)
---

# TUnit Development Guide for AI Assistants

## Table of Contents

- [Critical Rules - READ FIRST](#critical-rules---read-first)
- [Quick Reference](#quick-reference)
- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Development Workflow](#development-workflow)
- [Code Style Standards](#code-style-standards)
- [Testing Guidelines](#testing-guidelines)
- [Performance Requirements](#performance-requirements)
- [Common Patterns](#common-patterns)
- [Troubleshooting](#troubleshooting)

---

## Critical Rules - READ FIRST

### The Five Commandments

1. **DUAL-MODE IMPLEMENTATION IS MANDATORY**
   - Every feature MUST work identically in both execution modes:
     - **Source-Generated Mode**: Compile-time code generation via `TUnit.Core.SourceGenerator`
     - **Reflection Mode**: Runtime discovery via `TUnit.Engine`
   - Test both modes explicitly. Never assume parity without verification.
   - If you only implement in one mode, the feature is incomplete and MUST NOT be merged.

2. **SNAPSHOT TESTS ARE NON-NEGOTIABLE**
   - After ANY change to source generator output:
     ```bash
     dotnet test tests/TUnit.Core.SourceGenerator.Tests
     # Review .received.txt files, then:
     for f in *.received.txt; do mv "$f" "${f%.received.txt}.verified.txt"; done  # Linux/macOS
     for %f in (*.received.txt) do move /Y "%f" "%~nf.verified.txt"  # Windows
     ```
   - After ANY public API change (TUnit.Core, TUnit.Engine, TUnit.Assertions):
     ```bash
     dotnet test tests/TUnit.PublicAPI
     # Review and accept snapshots as above
     ```
   - Commit ALL `.verified.txt` files. These are the source of truth.

3. **NEVER USE VSTest APIs**
   - This project uses **Microsoft.Testing.Platform** exclusively
   - VSTest is legacy and incompatible with TUnit's architecture
   - If you see `Microsoft.VisualStudio.TestPlatform`, it's wrong

4. **PERFORMANCE IS A FEATURE**
   - TUnit is used by millions of tests daily
   - Every allocation in discovery/execution hot paths matters
   - Profile before and after for any changes in critical paths
   - Use `ValueTask`, object pooling, and cached reflection

5. **AOT/TRIMMING COMPATIBILITY IS REQUIRED**
   - All code must work with Native AOT and IL trimming
   - Use `[DynamicallyAccessedMembers]` and `[UnconditionalSuppressMessage]` appropriately
   - Test changes with AOT-compiled projects when touching reflection

---

## Quick Reference

### ⚠️ CRITICAL: TUnit.TestProject Testing Rules

**DO NOT run `TUnit.TestProject` without filters!** Many tests are intentionally designed to fail to verify error handling, diagnostics, and edge cases.

```bash
# ❌ WRONG - Will show many "failures" (this is expected behavior)
cd tests/TUnit.TestProject && dotnet run
cd tests/TUnit.TestProject && dotnet test

# ✅ CORRECT - Always use targeted filters when testing TUnit.TestProject
cd tests/TUnit.TestProject && dotnet test --treenode-filter "/*/*/SpecificClass/*"
cd tests/TUnit.TestProject && dotnet test --treenode-filter "/*/*/*/*[Category!=Performance]"

# ✅ CORRECT - Test other test projects normally (they don't have intentional failures)
dotnet test tests/TUnit.Engine.Tests
dotnet test tests/TUnit.Assertions.Tests
dotnet test tests/TUnit.Core.SourceGenerator.Tests
```

**Why TUnit.TestProject is special:**

- Contains negative test cases (tests that verify failures work correctly)
- Tests error messages, diagnostics, and exception handling
- Performance tests excluded by default
- Integration tests covering all edge cases

**Golden Rule**: When verifying changes, use `dotnet test` (which excludes TUnit.TestProject), OR run TUnit.TestProject with specific `--treenode-filter` patterns.

---

### Most Common Commands

```bash
# Run all tests (automatically excludes TUnit.TestProject integration tests)
dotnet test

# Test source generator + accept snapshots
dotnet test tests/TUnit.Core.SourceGenerator.Tests
for f in *.received.txt; do mv "$f" "${f%.received.txt}.verified.txt"; done

# Test public API + accept snapshots
dotnet test tests/TUnit.PublicAPI
for f in *.received.txt; do mv "$f" "${f%.received.txt}.verified.txt"; done

# Run specific test by tree node filter
dotnet test --treenode-filter "/Assembly/Namespace/ClassName/TestName"

# Run tests excluding performance tests
dotnet test --treenode-filter "/*/*/*/*[Category!=Performance]"

# Build in release mode
dotnet build -c Release

# Test AOT compilation
dotnet publish -c Release -p:PublishAot=true
```

### Snapshot Workflow Quick Ref

```
┌─────────────────────────────────────────────────────────┐
│ 1. Make change to source generator or public API       │
│ 2. Run relevant test: dotnet test [Project]            │
│ 3. If snapshots differ, review .received.txt files     │
│ 4. If changes are correct, rename to .verified.txt     │
│ 5. Commit .verified.txt files                          │
│ 6. NEVER commit .received.txt files                    │
└─────────────────────────────────────────────────────────┘
```

### Test Filter Syntax

```bash
# Single test
--treenode-filter "/TUnit.TestProject/Namespace/ClassName/TestMethodName"

# All tests in a class
--treenode-filter "/*/*/ClassName/*"

# Multiple patterns (OR logic)
--treenode-filter "Pattern1|Pattern2"

# Exclude by category

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thomhurst/TUnit](https://github.com/thomhurst/TUnit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
