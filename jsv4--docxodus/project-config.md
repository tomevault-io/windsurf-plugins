---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Important Instructions

- **Never credit yourself in commits.** Do not add "Generated with Claude Code" or "Co-Authored-By: Claude" to commit messages.

## Coding Standards

### Nullable Reference Types

The project has `<Nullable>disable</Nullable>` globally due to ~9,000 warnings in legacy code. However, **new code should use nullable annotations**:

- **New files**: Add `#nullable enable` at the top of the file
- **Substantial refactors**: When significantly modifying an existing file, consider adding `#nullable enable` and fixing warnings in that file
- **Use proper annotations**: Mark nullable parameters/returns with `?`, use null checks or `!` where appropriate

```csharp
#nullable enable

namespace Docxodus;

public class MyNewClass
{
    public string Name { get; set; } = string.Empty;  // Non-nullable with default
    public string? Description { get; set; }           // Explicitly nullable

    public string? FindItem(string key)                // May return null
    {
        // ...
    }
}
```

See [Issue #13](https://github.com/JSv4/Docxodus/issues/13) for the full nullable migration plan.

## Feature Development Workflow

When implementing new features or significant changes, follow this workflow:

### 1. Documentation Updates

- **CHANGELOG.md** - Add entry under `[Unreleased]` section describing the feature/fix
- **CLAUDE.md** - Update if the feature adds new settings, modules, or changes architecture
- **docs/architecture/** - Create or update architecture docs for significant features (e.g., `comment_rendering.md`, `comparison_engine.md`)
- **docs/ooxml_corner_cases.md** - Document any OOXML edge cases where Word's behavior differs from spec or our implementation (see below)

### 2. Test Updates

- Add tests to the appropriate test file in `Docxodus.Tests/`:
  - `HtmlConverterTests.cs` - WmlToHtmlConverter features
  - `WmlComparerTests.cs` - Document comparison features
  - `DocumentBuilderTests.cs` - Document merging/splitting
  - Use existing test files from `TestFiles/` when possible
  - When creating programmatic test documents, ensure all required parts exist (StyleDefinitionsPart, DocumentSettingsPart, etc.)

### 3. WASM/npm Wrapper Updates

Update these when adding new settings or methods to the .NET API:

- **wasm/DocxodusWasm/DocumentConverter.cs** - Add new JSExport methods or parameters
- **wasm/DocxodusWasm/DocumentComparer.cs** - For comparison-related changes
- **npm/src/types.ts** - Add TypeScript types, enums, and update `DocxodusWasmExports` interface
- **npm/src/index.ts** - Update wrapper functions to use new WASM methods

Build and verify with:
```bash
npm run build          # Builds WASM and TypeScript
dotnet test            # Run .NET tests
```

### 4. When to Update Each Layer

| Change Type | .NET | Tests | WASM | npm/TS | Docs |
|-------------|------|-------|------|--------|------|
| New converter setting | ✓ | ✓ | ✓ | ✓ | ✓ |
| Bug fix | ✓ | ✓ | - | - | CHANGELOG |
| New public enum | ✓ | ✓ | ✓ | ✓ | ✓ |
| Internal refactor | ✓ | ✓ | - | - | - |
| New module | ✓ | ✓ | ✓ | ✓ | ✓ |

## Build Commands

```bash
# Build the entire solution
dotnet build Docxodus.sln

# Build specific project
dotnet build Docxodus/Docxodus.csproj
```

## Test Commands

```bash
# Run all tests
dotnet test Docxodus.Tests/Docxodus.Tests.csproj

# Run a specific test by name
dotnet test --filter "FullyQualifiedName~DB001_DocumentBuilderKeepSections"

# Run tests for a specific test class
dotnet test --filter "FullyQualifiedName~DbTests"
```

## Architecture Overview

Docxodus is a library for manipulating Open XML documents (DOCX, XLSX, PPTX) built on top of the Open XML SDK. It is a fork of OpenXmlPowerTools upgraded to .NET 8.0. All code is in the `Docxodus` namespace.

### Document Wrapper Classes

The library uses in-memory byte array wrappers for documents:
- `DocxodusDocument` - Base class holding `DocumentByteArray` and `FileName`
- `WmlDocument` - Word documents (.docx)
- `SmlDocument` - Spreadsheet documents (.xlsx)
- `PmlDocument` - Presentation documents (.pptx)

These allow immutable-style document manipulation via `OpenXmlMemoryStreamDocument` pattern:
```csharp
using (OpenXmlMemoryStreamDocument streamDoc = new OpenXmlMemoryStreamDocument(doc))
{
    using (WordprocessingDocument document = streamDoc.GetWordprocessingDocument())
    {
        // modify document
    }
    return streamDoc.GetModifiedWmlDocument();
}
```

### Core Modules

**DocumentBuilder.cs** - Merge/split DOCX files. Uses `Source` objects to specify document ranges:
```csharp
var sources = new List<Source> { new Source(wmlDoc, keepSections: true) };
DocumentBuilder.BuildDocument(sources, outputPath);
```

**WmlComparer.cs** - Compare two DOCX files, producing a document with tracked revisions. Supports nested tables and text boxes. Key settings in `WmlComparerSettings`:
- `AuthorForRevisions` - Author name for tracked changes
- `DetailThreshold` - 0.0-1.0, lower = more detailed comparison (default: 0.15)
- `CaseInsensitive` - Case-insensitive comparison
- `DetectMoves` - Enable move detection in `GetRevisions()` (default: true)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JSv4/Docxodus](https://github.com/JSv4/Docxodus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
