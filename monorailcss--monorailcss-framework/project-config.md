---
trigger: always_on
description: MSBuild task library for integrating MonorailCss into .NET build pipelines. Provides build-time CSS generation by scanning project files for utility classes.
---

# MonorailCss.Build.Tasks

MSBuild task library for integrating MonorailCss into .NET build pipelines. Provides build-time CSS generation by scanning project files for utility classes.

## Purpose

This project packages MonorailCss as an MSBuild task that:
1. Scans content files (Razor, HTML, JSX, etc.) for utility class names
2. Parses input CSS files for theme variables, custom utilities, and component rules
3. Supports Tailwind v4 `@source` and `@import` directives for flexible source configuration
4. Generates optimized CSS output using the MonorailCss framework
5. Runs automatically during build, ensuring CSS stays synchronized with markup

## Key Components

### ProcessCssTask (`ProcessCssTask.cs`)
Main MSBuild task that orchestrates the build process. Accepts:
- `InputFile`: CSS file with theme definitions, custom utilities, and source directives (e.g., `app.css`)
- `OutputFile`: Generated CSS output path

**Incremental Build Support**: The task implements timestamp-based incremental builds. If the output file exists and is newer than the input file, CSS regeneration is skipped. This prevents duplicate file tracking errors from the Static Web Assets system and improves build performance.

Source scanning is controlled entirely through `@source` and `@import` directives in the input CSS file.

Uses regex patterns to extract class names from various frameworks:
- `class="..."` - HTML/Razor
- `className="..."` - React/JSX
- `:class="..."` - Vue
- `@class(...)` - Blazor
- `classList={...}` - Solid/object syntax

### Parsing (`Parsing/`)

**CssThemeParser.cs**: Parses input CSS files for:
- `@theme` blocks containing CSS custom properties
- Component rules with `@apply` directives
- `@utility` blocks for custom utility definitions
- `@import` and `@source` directives for source configuration

**CssSourceParser.cs**: Parses Tailwind v4 source and import directives:
- `@import "path"` - Import CSS files (parsed but not yet processed)
- `@import "path" source("path")` - Set base path for auto-detection
- `@import "path" source(none)` - Disable auto-detection
- `@import "path" theme(static)` - Theme modifier (parsed but not yet implemented)
- `@import "path" layer(utilities)` - Layer modifier (parsed but not yet implemented)
- `@source "path"` - Explicitly include a file or directory
- `@source not "path"` - Exclude a file or directory
- `@source inline("pattern")` - Safelist utilities with brace/variant expansion
- `@custom-variant name (selector)` - Define custom variant with selector pattern

**CustomUtilityCssParser.cs**: Extracts custom utility definitions from `@utility` directives, supporting:
- Static utilities (e.g., `@utility scrollbar-none`)
- Wildcard patterns (e.g., `@utility scrollbar-thumb-*`)
- `@apply` directives within utilities (e.g., `@utility bordered-link { @apply font-semibold border-b; }`)
- Nested selectors with `&` references
- Automatic custom property dependency tracking

**SourceConfiguration.cs**: DTOs representing parsed source directives including include/exclude paths, inline utilities, and base path configuration.

**PathPlaceholderResolver.cs**: Resolves dynamic placeholders in file paths using MSBuild properties. Supports `$(Configuration)`, `$(TargetFramework)`, and `$(RuntimeIdentifier)` placeholders with case-insensitive matching. Enables build-configuration-agnostic `@source` paths.

### Scanning (`Scanning/`)

**GlobScanner.cs**: Handles glob pattern matching for file system scanning. Supports:
- Wildcards (`*`) for single-level matching
- Recursive globs (`**`) for multi-level directory traversal
- Brace expansion (`{Pages,Components}` or `{razor,cs}`) for multiple alternatives
- Automatic exclusion of `bin` and `obj` directories
- Cross-platform path normalization

**DllScanner.cs**: Scans .NET assemblies (DLLs) for utility class strings embedded in string literals using PE metadata reader. Extracts string values from the #Strings heap in the metadata tables.

### Static Web Asset Scanning

Component packages (RCLs) ship JS under `_content/<Package>/` (e.g. `_content/Pennington.UI/scripts.js`) that builds markup at runtime. Those files live in the NuGet cache, never enter IL, and sit outside the project tree, so neither the content sweep nor the DLL scan reaches them. The `targets` file resolves them via `ResolveBuildStaticWebAssets` (which populates `@(StaticWebAsset)` with own + project + package assets), filters to `.js`/`.mjs` (the `AssetRole != Alternative` filter drops compressed `.gz`/`.br` siblings), and forwards the physical paths to `ProcessCssTask.StaticWebAssets`. They flow through the same `SourceFileScanner` as content files (generic-tokenizer strategy for `.js`). The dependency is gated on `'$(UsingMicrosoftNETSdkRazor)' == 'true'` so plain library/console consumers build untouched; set `MonorailCssScanStaticWebAssets=false` to opt out. Assets owned by a `MonorailCssExcludeAssembly` are skipped by their `SourceId` metadata, mirroring the runtime's `_content/<Package>` exclusion. Mirrors `MonorailDiscoveryOptions.ScanStaticWebAssets` on the runtime side.

## Usage


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [monorailcss/MonorailCss.Framework](https://github.com/monorailcss/MonorailCss.Framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
