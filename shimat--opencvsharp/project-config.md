---
trigger: always_on
description: All text files in this repository use **UTF-8 without a BOM**. This applies to `.cs`, `.csproj`, `.yml`, `.md`, `.json`, `.cpp`, `.h`, and every other text file — there are no exceptions.
---

# Copilot Instructions

## File encoding

All text files in this repository use **UTF-8 without a BOM**. This applies to `.cs`, `.csproj`, `.yml`, `.md`, `.json`, `.cpp`, `.h`, and every other text file — there are no exceptions.

- `.editorconfig` declares `charset = utf-8` (no BOM) for all files, so editors and `dotnet format` enforce this automatically.
- The `Write` / `create_file` tool already produces UTF-8 without a BOM, so just create and edit files normally — no encoding fix-up step is needed.
- Do **not** add a BOM, and do not save as ANSI, Shift-JIS, or UTF-16. The repository is English-only (ASCII), and the C++ build sets the UTF-8 source charset, so MSVC reads the sources correctly without a BOM.
- **All repository content is English**, regardless of what language a request or discussion happens in: code, comments, identifiers, commit messages, PR/issue titles and descriptions, and every file under `docs/`. This holds even when the person or agent driving the change is working with the maintainer in a different language.

## NuGet README sync

When editing the root `README.md`, also update the NuGet-specific README files accordingly.

| File | Target packages |
|---|---|
| `packaging/nuget/README.managed.md` | `OpenCvSharp5`, `OpenCvSharp5.Windows`, `OpenCvSharp5.Windows.Slim`, `OpenCvSharp5.GdipExtensions`, `OpenCvSharp5.WpfExtensions`, `OpenCvSharp5.AvaloniaExtensions` |
| `packaging/nuget/README.runtime.md` | `OpenCvSharp5.runtime.*`, `OpenCvSharp5.official.runtime.*` (all native runtime packages) |

> The `main`-branch packages are the `OpenCvSharp5.*` family (OpenCV 5.x). An identically-structured `OpenCvSharp4.*` family (OpenCV 4.13.0, frozen on the `4.x` branch) is also published for .NET Framework / pre-.NET 8 consumers — see the branch note under "Versioning and release process".

The NuGet READMEs are a subset of the top-level README and consist of the following sections:

- Overview and supported platforms
- Installation instructions (Quick Start)
- Requirements
- Slim profile module coverage table (`README.managed.md` only)
- Code usage examples (`README.managed.md` only)
- Links (GitHub, Samples, API Docs, Issue Tracker)

Do **not** include CI badges, Docker instructions, build instructions, or donation links in the NuGet READMEs.

## Markdown authoring

Do not hard-wrap Markdown prose at a fixed character width in the middle of a sentence. Write each paragraph, and each list item, as a single unbroken line in the source, and let the renderer/editor soft-wrap it for display. Only start a new line for a genuinely new block: the next list item, a new paragraph, a heading, a code fence, etc. This applies to every Markdown file in the repo (READMEs, `docs/*.md`, this file) as well as PR/issue/commit-message bodies that render as Markdown — manually wrapping mid-sentence makes diffs noisy (a small wording edit reflows the whole paragraph) and reads as pointlessly rigid formatting. When editing an existing hard-wrapped file, prefer reflowing the touched paragraphs to single lines rather than preserving the old wrap width.

## Agent mode — terminal commands

In agent mode, do **not** use display commands that require user input (e.g., `more`, `less` without options). Use non-interactive alternatives instead:
- PowerShell: `Select-Object -First N`, `Out-String`, `Write-Output`
- Git: pass `-P` or `--no-pager`, or pipe to `Out-String`; e.g. `git --no-pager diff`
- Use `cat` for displaying file contents.

## Adding a new OpenCV class wrapper

> **Scope**: This checklist covers `cv::SomeClass : cv::Algorithm` subclasses. OpenCV also has classes that do **not** inherit from `Algorithm` — those may follow different ownership and lifetime patterns (see existing non-Algorithm wrappers such as `BackgroundSubtractor` or classes in `core/` for reference).

Follow this checklist when wrapping a new `cv::SomeClass : cv::Algorithm` class:

### Files to create

| File | Location |
|---|---|
| `<module>_SomeClass.h` | `src/OpenCvSharpExtern/` |
| `SomeClass.cs` | `src/OpenCvSharp/Modules/<module>/` |
| `NativeMethods_<module>_SomeClass.cs` | `src/OpenCvSharp/Internal/PInvoke/NativeMethods/<module>/` |
| `SomeClassTest.cs` | `test/OpenCvSharp.Tests/<module>/` |
| `Enum/SomeEnum.cs` (if needed) | same module folder |
| `VectorOfVecXy.cs` (only if the element type is non-blittable/nested — blittable types use `StdVector<T>` directly, no new file needed) | `src/OpenCvSharp/Internal/Vectors/` |

### Files to modify

| File | Change |
|---|---|
| `<module>.cpp` | Add `#include "<module>_SomeClass.h"` |
| `std_vector.h` | Add `#pragma region cv::VecXy` block if new vector type needed |
| `NativeMethods_stdvector.cs` | Add corresponding P/Invoke region if new vector type needed |
| `Cv<Module>.cs` | Add `public static SomeClass CreateSomeClass()` factory method |

### C++ extern pattern (<module>_SomeClass.h)

```cpp
#pragma once
#ifndef NO_CONTRIB
#include "include_opencv.h"

CVAPI(ExceptionStatus) <module>_Ptr_SomeClass_delete(cv::Ptr<cv::<module>::SomeClass> *obj)
{ BEGIN_WRAP delete obj; END_WRAP }

CVAPI(ExceptionStatus) <module>_Ptr_SomeClass_get(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shimat/opencvsharp](https://github.com/shimat/opencvsharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
