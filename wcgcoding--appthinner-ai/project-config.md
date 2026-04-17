---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```bash
# Build (Debug)
xcodebuild -project AppThinnerAnalyzer/AppThinnerAnalyzer.xcodeproj \
  -scheme AppThinnerAnalyzer -destination 'platform=macOS' -configuration Debug build

# Build (Release)
xcodebuild -project AppThinnerAnalyzer/AppThinnerAnalyzer.xcodeproj \
  -scheme AppThinnerAnalyzer -destination 'platform=macOS' -configuration Release build

# Run all tests
xcodebuild test -project AppThinnerAnalyzer/AppThinnerAnalyzer.xcodeproj \
  -scheme AppThinnerAnalyzer -destination 'platform=macOS'
```

The project has no linter configuration (no `.swiftlint.yml`). Swift package dependencies (MachOKit, MachOObjCSection, etc.) are resolved automatically by Xcode.

## Architecture Overview

This is a **macOS SwiftUI app** (macOS 14.0+) for analyzing iOS app package sizes. It follows MVVM with a protocol-based dependency injection container.

### Three Data Sources → Unified TreeMap

The entire application is built around fusing three independent data sources into a single file tree keyed by **project-relative paths**:

| Source | Parser | Output | Use |
|--------|--------|--------|-----|
| LinkMap `.txt` | `LinkmapAnalyzer` | `CodeSizeInfo[]` — per-`.o` code size mapped to project path | `codeSize` on each tree node |
| IPA / `.app` | `PackageParser` | `PackageFileInfo[]` — package-relative paths + sizes | `resourceSize` / `frameworkSize` |
| Project directory | `ProjectResourceScanner` | `ProjectFileEntry[]` — all source & resource files | Tree skeleton; path index for LinkMap mapping |

The **bridge key** between all three is the project-relative path (`relativePath`). Every matching step ultimately resolves to this shared key.

### Analysis Pipeline (`AnalysisService.analyzeProject`)

```
Phase 1 (concurrent): IPA parse + project scan → then LinkMap (depends on project entries)
Phase 2: buildIntegratedDataFromProjectEntries → IntegratedAnalysisData (codeSize + resourceSize + frameworkSize per file)
Phase 3: buildUnusedContentResults → BinaryUnusedCodeAnalyzer (Mach-O) + UnusedScanService (resources)
Phase 4: createAnalysisProject → CoreData write (AnalysisProject + AnalysisResult rows)
```

`AnalysisService` is the sole orchestrator. All other services are called from it and should not call each other directly.

### LinkMap Path Resolution (`LinkmapPathAdapter`)

LinkMap object file paths (absolute, machine-specific) are resolved to project-relative paths in priority order:

1. **Static lib** (`.a(Obj.o)` or `.a[N](Obj.o)`) → `staticLibLookup["pathComponent|baseName"]` or `staticLibNameToPath[libName]`
2. **Framework** (`.framework/Name(Obj.o)`) → `frameworkNameToPath[fwName]` + append original `fileName` to produce a **virtual `.o` path** like `Pods/X.framework/X(Foo.o)`
3. **Base name** → `projectFileIndex[baseName]`
4. **Fallback** → stripped linkmap path (logged as unmapped)

Step 2 preserves the parenthesized filename so that `AnalysisService.isVirtualObjectPath()` can recognize it and create per-compilation-unit virtual nodes in the tree, instead of collapsing everything to the `.framework` directory. This is the key mechanism for showing `.framework` contents at `.o` granularity.

### Virtual `.o` Nodes

When a LinkMap path resolves via the framework step, `codeSizeInfo.relativePath` looks like `Pods/SDWebImage/SDWebImage.framework/SDWebImage(SDImageCache.o)`. In `buildIntegratedDataFromProjectEntries`:

- `isVirtualObjectPath(path)` detects these (contains `(` and ends with `.o)`)
- They are split from real source entries and become independent `IntegratedFileInfo` nodes
- The `.framework` container's `frameworkSize` is zeroed to avoid double-counting
- In `createAnalysisProject`, `isUnusedCode` for virtual nodes is determined by `virtualNodeContainsUnusedClass()` (matching the `.o` base name against the unused class name set), not by path lookup

### Unused Detection

- **Unused code**: `BinaryUnusedCodeAnalyzer` parses the main binary's `__objc_classlist` (defined classes) minus `__objc_classrefs` + `superClassName` + category host classes → candidates filtered by a whitelist of runtime-dynamic suffixes (`ViewController`, `Delegate`, `Cell`, etc.)
- **Unused resources**: `UnusedScanService` collects resource name references from source files via regex, then checks each resource file against this set
- Both run inside `UnusedScanService.runLocalUnusedScan`, called from `AnalysisService`

The `MachOKit` and `MachOObjCSection` dependencies are optional; both are guarded with `#if canImport(MachOKit)`. Without them the binary analysis step is skipped silently.

### CoreData Schema

Two main entities under `AnalysisProject` (1-to-many):
- **`AnalysisResult`**: one row per file/virtual node — `relativePath`, `codeSize`, `resourceSize`, `frameworkSize`, `isUnusedCode`, `isUnusedResource`
- **`ExternalUnusedData`**: imported external unused lists

`AnalysisProject.analysisResultsArray` sorts by `relativePath` and is the sole input to `TreemapGenerator`.

### TreeMap Generation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wcgCoding) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
