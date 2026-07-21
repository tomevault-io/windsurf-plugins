---
trigger: always_on
description: SkiaSharp is a cross-platform 2D graphics API for .NET wrapping Google's Skia library.
---

# SkiaSharp

SkiaSharp is a cross-platform 2D graphics API for .NET wrapping Google's Skia library.

**Architecture:** `C# Wrapper` -> `P/Invoke` -> `C API` -> `C++ Skia`
**Principle:** C# validates parameters, C API trusts and passes through.

---

## Critical Rules (Read First)

These rules are **non-negotiable**. Violating them causes broken builds, crashes, or downstream breakage.

### 1. Bootstrap First

Before C# code can build, native binaries must exist in `output/native/`. **How** you produce them depends on what you're changing:

| You are changing… | Bootstrap with |
|---|---|
| **Only C# code** (no files under `externals/skia/`, no `DEPS`, no submodule bump) | `dotnet cake --target=externals-download` (downloads pre-built natives from the **current** milestone) |
| **Native code, C API, `DEPS`, or the Skia submodule** (incl. milestone updates) | `dotnet cake --target=externals-{platform} --arch={arch}` — build from source. |

> **🛑 If you are doing a Skia milestone update, a C API change, or anything under `externals/skia/`, STOP. Do not run `externals-download` — ever. The downloaded binaries are from the OLD milestone and do not contain your changes; using them produces silently-wrong builds and `EntryPointNotFoundException` at runtime.** When source builds fail (missing `gn`, network errors, etc.), debug the source build — do not fall back to download.

### 2. Never `externals-download` After Native Changes

If you have modified **any** of the following, `externals-download` is FORBIDDEN until your changes ship to the pre-built artifact server (which only happens after merge):

- `externals/skia/**` (including the submodule SHA)
- `externals/skia/src/c/**`, `externals/skia/include/c/**`
- `externals/skia/DEPS`
- Any milestone bump or version file (`VERSIONS.txt`, `sk_types.h SK_C_INCREMENT`)

Falling back to `externals-download` because a native build failed is the #1 way agents corrupt milestone updates. Fix the source build instead.

### 3. Never Edit Generated Files

Files matching `*.generated.cs` and `docs/` are auto-generated.

- **NEVER** manually edit these files
- **ALWAYS** regenerate after C API changes (see [Commands](#commands))

### 4. ABI Stability

SkiaSharp maintains stable ABI. Breaking changes break downstream apps.

| Allowed | Never |
|---------|-------|
| Add new overloads | Modify existing signatures |
| Add new methods | Remove public APIs |
| Add new classes | Change return types |

### 5. Tests Are Mandatory

**Building alone is NOT sufficient.** Run tests before claiming completion (see [Commands](#commands)).

### 6. Branch Protection (COMPLIANCE REQUIRED)

**Direct commits to protected branches are a policy violation.**

| Repository | Protected Branches |
|------------|-------------------|
| SkiaSharp (parent) | `main` |
| externals/skia (submodule) | `main`, `skiasharp` |

**Required workflow:**

1. **Create a feature branch FIRST** — Use naming convention: `dev/issue-NNNN-description`
2. **Make all commits on the feature branch** — Never commit directly to protected branches
3. **Submit a Pull Request** — Changes must be reviewed before merging

```bash
# CORRECT — Always create a feature branch first
git checkout -b dev/issue-1234-fix-description

# For submodule changes:
cd externals/skia
git checkout -b dev/issue-1234-add-c-api

# NEVER DO THIS — Policy violation
git checkout main && git commit  # FORBIDDEN
git checkout skiasharp && git commit  # FORBIDDEN (in skia submodule)
```

**This applies to BOTH repositories.** The skia submodule has its own protected branches that must be respected.

---

## Commands

Single source of truth for all commands:

| Task | Command |
|------|---------|
| **Bootstrap (C#-only work — see Rule #1, FORBIDDEN for native changes)** | `dotnet cake --target=externals-download` |
| **Build Native (macOS ARM64)** | `dotnet cake --target=externals-macos --arch=arm64` |
| **Build Native (macOS Intel)** | `dotnet cake --target=externals-macos --arch=x64` |
| **Build Native (Windows x64)** | `dotnet cake --target=externals-windows --arch=x64` |
| **Build Native (Linux x64)** | `dotnet cake --target=externals-linux --arch=x64` |
| **Build Native (Linux ARM64)** | `dotnet cake --target=externals-linux --arch=arm64` |
| **Build C#** | `dotnet build binding/SkiaSharp/SkiaSharp.csproj` |
| **Test** | `dotnet test tests/SkiaSharp.Tests.Console/SkiaSharp.Tests.Console.csproj` |
| **Regenerate** | `pwsh ./utils/generate.ps1` |
| **Regenerate release-notes (Prepare: api diffs + facts + index)** | `.agents/skills/release-notes/scripts/prepare.sh [--force] [--min-version X --max-version Y]` |
| **Render all pages + TOC/index (offline, from committed JSON)** | `.agents/skills/release-notes/scripts/render.sh [--min-version X --max-version Y]` |

### When to Use Which Bootstrap

| What You Changed | Command Required |
|------------------|------------------|
| C# code only (`binding/SkiaSharp/*.cs`) | `externals-download` (pre-built natives) |
| C API (`externals/skia/src/c/`, `externals/skia/include/c/`) | **`externals-{platform}` (MUST rebuild natives — `externals-download` is FORBIDDEN)** |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mono/SkiaSharp](https://github.com/mono/SkiaSharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
