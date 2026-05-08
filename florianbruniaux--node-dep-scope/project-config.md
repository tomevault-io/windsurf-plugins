---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**dep-scope** is a granular dependency analyzer for TypeScript/JavaScript projects. Unlike Knip or Depcheck (binary used/unused), it provides symbol-level analysis: which symbols from each dependency are used, usage percentages, native alternatives, duplicate detection, and peer dependency identification.

**V2 feature**: `dep-scope migrate` generates LLM-ready markdown migration prompts for dependencies that can be removed. Run without arguments to auto-detect all candidates; pass a package name to target one. Prompts are context-aware (TypeScript target, framework, file locations) and ready to pipe into Claude Code.

**e18e integration**: 169 packages from the [e18e module-replacements](https://e18e.dev) project are embedded statically in `src/rules/e18e-data.ts`. Any import from a recognized micro-utility or polyfill package (e.g. `has-flag`, `left-pad`, `array-includes`, `object-assign`) is automatically flagged as RECODE_NATIVE with the native replacement shown. Total packages with alternatives: 195.

## Commands

```bash
npm run build      # Compile TypeScript to dist/
npm run dev        # Watch mode compilation
node dist/cli/index.js --help  # Run CLI directly

# CLI usage
dep-scope init                                # Interactive wizard: detect project + generate config
dep-scope init --yes                          # Non-interactive: write defaults (CI-safe)
dep-scope scan                                # Scan all dependencies
dep-scope scan --root                         # Scan full project root (scripts/, tools/, bin/, etc.)
dep-scope scan --check-duplicates             # Include duplicate detection
dep-scope analyze <package>                   # Analyze specific package
dep-scope analyze <package> --root            # Analyze with full project scan scope
dep-scope duplicates                          # Find duplicate libraries
dep-scope report -p /path -o ./audit.md       # Generate full report
dep-scope migrate                             # Auto-detect + generate migration prompts
dep-scope migrate <package>                   # Generate migration prompt for one package
dep-scope migrate --dry-run                   # Preview without writing files
```

## Architecture

```
src/
├── types/index.ts              # Core types: Verdict, DependencyAnalysis, InvestigateReason
├── analyzers/
│   ├── import-analyzer.ts      # AST parsing via @typescript-eslint/parser
│   ├── usage-analyzer.ts       # Main orchestrator - aggregates imports, determines verdicts
│   └── peer-dep-analyzer.ts    # Scans node_modules/*/package.json for peer deps
├── cli/
│   ├── index.ts                # Commander.js CLI entry point
│   └── init-wizard.ts          # Interactive wizard (@inquirer/prompts) for dep-scope init
├── config/
│   ├── schema.ts               # Zod schemas for config validation
│   ├── loader.ts               # Multi-format config loading (JSON, YAML, TS, JS)
│   ├── defaults.ts             # DEFAULT_WELL_KNOWN_PATTERNS, default values
│   ├── presets/                # Built-in presets (minimal, react, node)
│   └── index.ts                # Config exports + defineConfig helper
├── migration/
│   ├── types.ts                # MigrationTemplate, MigrationContext, MigrationOutput
│   ├── generator.ts            # Prompt engine: DependencyAnalysis + Context + Template → markdown
│   ├── templates/
│   │   ├── lodash.ts           # Hand-crafted template (12 symbols + catch-all)
│   │   ├── moment.ts           # Hand-crafted template (11 symbols: format, add, diff, parse...)
│   │   └── axios.ts            # Hand-crafted template (8 symbols: get, post, put, interceptors...)
│   └── index.ts                # Public API + template registry + buildGenericTemplate()
├── rules/
│   ├── native-alternatives.ts  # Maps library symbols to native JS alternatives + e18e integration
│   ├── e18e-data.ts            # 169 packages from e18e/module-replacements (static, no runtime dep)
│   ├── duplicate-categories.ts # Defines functional overlaps (icons, date, http, etc.)
│   └── well-known-packages.ts  # Pattern matching for auto-KEEP/IGNORE
├── reporters/
│   ├── console-reporter.ts     # Terminal output with picocolors
│   └── markdown-reporter.ts    # Markdown report generation
├── utils/
│   ├── path-alias-detector.ts  # Filters @/, ~/, tsconfig paths from analysis
│   ├── project-detector.ts     # Detects framework, existing dirs, preset — used by init wizard
│   ├── src-paths-resolver.ts   # Auto-detects source directories when configured paths don't exist
│   └── tsconfig-resolver.ts    # Resolves compilerOptions.target following extends chains
└── index.ts                    # Public API exports
```

### Data Flow

1. CLI loads config via `loadConfig()` → detects .depscoperc, YAML, TS, package.json#depScope
2. Config merged: Defaults → Presets (extends) → File Config → CLI Options via `resolveConfig()`
3. `resolveSrcPaths()` resolves source directories — falls back to auto-detection if configured paths don't exist
4. `UsageAnalyzer.scanProject()` reads package.json dependencies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FlorianBruniaux/node-dep-scope](https://github.com/FlorianBruniaux/node-dep-scope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
