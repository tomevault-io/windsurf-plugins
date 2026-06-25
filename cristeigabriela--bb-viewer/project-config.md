---
trigger: always_on
description: Instructions for AI assistants working on this project.
---

# CLAUDE.md

Instructions for AI assistants working on this project.

## What is bb-viewer?

A vanilla TypeScript SPA that visualizes Windows SDK and PHNT header analysis data produced by [bb](https://github.com/cristeigabriela/bb) v0.3.2+. It shows functions (with ABI layouts, parameter details, MSDN metadata, kernel/driver metadata including IRQL), types (struct **and union** with memory layouts, field tables, nested expansion, anonymous-record inline expansion), typedefs (as first-class navigable entries with chain resolution), constants/enums (including the full `STATUS_*` set from `ntstatus.h`), and a type relationship graph.

Built with Bun. No framework — plain DOM manipulation with a hash-based router.

## Project structure

```
bb-viewer/
├── src/
│   ├── main.ts              # Entry point, route registration
│   ├── router.ts            # Hash-based SPA router
│   ├── data.ts              # Data loading, indexing, xref building, resolver
│   ├── types.ts             # TypeScript interfaces for all data
│   ├── dom.ts               # Core DOM primitives ($, $$, el, clear)
│   ├── utils.ts             # matchQuery (glob/regex), debounce
│   ├── primitives.ts        # KNOWN_PRIMITIVES set — TRUE C primitives only
│   │                        # (Win32 typedefs like HANDLE/DWORD/LPCWSTR are in
│   │                        # typedefsByName, not here)
│   ├── irql.ts              # IRQL filter parsing + range-based matching
│   ├── theme.ts             # Dark/light mode + accent color picker
│   ├── dataset-switcher.ts  # WinSDK/PHNT + architecture + mode switching
│   ├── search-modal.ts      # Global search modal with preview pane
│   ├── clippy.ts            # ASCII art clippy popup (cowsay)
│   ├── ui/
│   │   ├── links.ts         # typeLink, funcLink, enumLink, badge, renderTypeStr, highlightCode
│   │   └── filter-dropdown.ts  # Checkbox filter dropdown widget
│   └── views/
│       ├── shared.ts        # Shared view helpers (sort row, search input, filter chips, pagination, collapsible sections, not-found)
│       ├── home.ts          # Dashboard with stat cards and bar charts
│       ├── functions.ts     # Function list + detail views (kernel filters live here)
│       ├── types.ts         # Type list + detail views — dispatches between record and typedef detail
│       ├── constants.ts     # Constants/enums list + detail views
│       ├── type-graph.ts    # Cytoscape.js type relationship graph
│       └── lookup.ts        # Universal /q/:name lookup
├── public/
│   ├── index.html           # SPA shell
│   ├── styles.css           # All CSS (terminal theme, dark/light, accent colors, IRQL chips)
│   └── app.js               # Built output (generated)
├── test/
│   └── irql.test.ts         # Bun unit tests for IRQL range semantics
├── data/                    # Generated JSON data
│   ├── {winsdk,phnt}[-kernel]/
│   │   └── {amd64,x86,arm,arm64}/
│   │       ├── funcs.json     # Func[] with .driver, .metadata.source
│   │       ├── types.json     # types[] + referenced_types[] (anon) + typedefs[]
│   │       ├── consts.json
│   │       └── graph.json     # Precomputed type graph with positions
├── build.ts                 # Bun bundler config
├── build-graph.ts           # Precomputes type relationship graph (d3-force)
├── serve.ts                 # Dev server with file watching
├── generate-data.ps1        # PowerShell script to regenerate all data
└── package.json
```

## Building

```powershell
bun install
bun run build         # bundle src/ → public/app.js
bun run build:graph   # precompute graph.json files (needs data/ populated)
bun run dev           # dev server with auto-rebuild on localhost:3000
bun test              # run unit tests (currently IRQL parsing/matching)
```

## Generating data

Requires bb v0.3.2+ binaries and Windows SDK installed. Kernel mode additionally needs the WDK (`winget install --exact --id Microsoft.WindowsWDK.10.0.26100`).

```powershell
.\generate-data.ps1                                    # all datasets, all archs, all modes
.\generate-data.ps1 -Dataset phnt                      # only phnt
.\generate-data.ps1 -Arch amd64                        # only amd64
.\generate-data.ps1 -Mode kernel                       # only kernel mode
.\generate-data.ps1 -BbBinDir 'D:\dev\rust\bb\bb\target\debug'  # use local binaries
```

The script auto-detects the Windows SDK path. After generating data, also run `bun run build:graph` to update the type graphs.

**`--struct '*'` is mandatory for bb-types**: without it the `typedefs[]` array only contains record-targeting typedefs (struct/union). Pointer typedefs (HANDLE → void *), primitive typedefs (DWORD → unsigned long), enum typedefs (FILE_INFORMATION_CLASS → _FILE_INFORMATION_CLASS), function-pointer typedefs, and array typedefs are dropped. The script handles this — but if you call bb-types directly, remember the flag.

Note: `bb-funcs --arch arm` and `bb-funcs --arch arm64` will fail (ARM ABI not yet implemented in bb). Types and constants work for all architectures.

## Data flow

1. bb parses C/C++ headers via libclang → outputs JSON (`types.json` has top-level `types`, `referenced_types`, **`typedefs`**)
2. `generate-data.ps1` runs bb for each dataset/arch/mode combo → `data/{dataset}[-kernel]/{arch}/*.json`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cristeigabriela/bb-viewer](https://github.com/cristeigabriela/bb-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
