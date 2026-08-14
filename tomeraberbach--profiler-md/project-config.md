---
trigger: always_on
description: A TypeScript package that converts performance profiles to human and LLM
---

# profiler-md

A TypeScript package that converts performance profiles to human and LLM
friendly Markdown.

## Project structure

```
profiler-md
├── src/
│   │
│   ├── index.ts                  # API entry point
│   │
│   ├── cli/
│   │   ├── index.ts              # CLI entry point that orchestrates the run
│   │   ├── cli.ts                # Optique flag/usage/topic definitions
│   │   ├── input.ts              # Reads stdin or file, decompresses gzip/brotli
│   │   ├── options.ts            # Builds API options from CLI flags
│   │   ├── output.ts             # Writes Markdown to file or stdout (optionally paged)
│   │   ├── pager.ts              # Spawns $PAGER or `less` for stdout output
│   │   ├── highlight.ts          # ANSI Markdown syntax highlighting for stdout
│   │   ├── theme-kindling.ts     # Custom Shiki theme for syntax highlighting
│   │   ├── logo.ts               # ASCII art logo printed to stderr by --version
│   │   ├── ansis.ts              # ANSI color helpers (respects TTY/no-color)
│   │   ├── help.ts               # Prints CLI help and per-topic docs
│   │   ├── languages.ts          # Language display metadata
│   │   ├── examples.ts           # Parses metadata from examples/ filenames
│   │   └── error.ts              # CliError class and top-level error reporting
│   │
│   ├── formats/                  # Individual profile format implementations
│   │   ├── converter.ts          # Format converter types
│   │   ├── registry.ts           # Format converter registry
│   │   ├── index.ts              # profileToMd(Async)/diffProfiles(Async) and format auto-detection
│   │   ├── **/<name>/            # One per format, top-level (e.g. collapsed) or nested in a subdirectory (e.g. v8/cpu-profile)
│   │   │   ├── matches.ts        # Cheap auto-detection check for the format
│   │   │   ├── parse.ts          # Parses input into a modality's parsed type
│   │   │   ├── index.ts          # Exports the format's converter
│   │   │   └── testing.ts        # Test-only utilities specific to this format (optional)
│   │   └── testing.ts            # Test-only utilities for running a converter and reading example inputs
│   │
│   ├── origins/                  # Profiler detection and categorization
│   │   ├── origin.ts             # OriginSpec type + match and frame-normalization helpers
│   │   ├── categorize.ts         # Generic categorization rule helpers
│   │   ├── jvm.ts                # JVM runtime conventions shared across origins
│   │   ├── javascript.ts         # JavaScript ecosystem conventions shared across origins
│   │   ├── cpython.ts            # CPython interpreter conventions shared across origins
│   │   ├── specs/
│   │   │   ├── <name>.ts         # One file per origin (e.g. node, node-pprof, jdk)
│   │   │   └── index.ts          # Exports originSpecs in detection-priority order
│   │   ├── index.ts              # Origin registry and derived detector
│   │   └── testing.ts            # Test-only origin detection and entry construction helpers
│   │
│   ├── modalities/               # Individual modality implementations
│   │   ├── aggregator.ts         # Uniform per-input aggregator contract all modalities implement
│   │   ├── diff.ts               # Base/current diffing primitives
│   │   ├── stack-frame.ts        # Stack frame type, distinct-frame origin detection, and normalization shared across modalities
│   │   ├── metric.ts             # Recorded metric types and inference logic
│   │   ├── measure.ts            # Metric phrasing and cell formatting shared across modalities
│   │   ├── table.ts              # Table cell/column types + Markdown table/diff-table formatting
│   │   ├── format.ts             # Formatting helpers shared across modalities
│   │   ├── call-stack-profile/   # Common call stack profile conversion logic
│   │   │   ├── type.ts           # Parsed call stack profile types
│   │   │   ├── aggregate.ts      # Observation aggregation over frames
│   │   │   ├── diff.ts           # Aggregated call stack profile diffing logic
│   │   │   ├── measure.ts        # Profile-resolved measure views with count fallback
│   │   │   ├── table.ts          # The call stack profile formatter's table columns
│   │   │   ├── format.ts         # Call stack profile and diff to Markdown formatting
│   │   │   ├── index.ts          # Barrel file
│   │   │   └── testing.ts        # Test-only utilities specific to this module
│   │   ├── call-graph/           # Common weighted call graph conversion logic
│   │   │   ├── type.ts           # Parsed call graph types
│   │   │   ├── aggregate.ts      # Function-node merging, cycle-safe totals, and categorization
│   │   │   ├── diff.ts           # Aggregated call graph diffing logic
│   │   │   ├── table.ts          # The call graph formatter's table columns
│   │   │   ├── format.ts         # Call graph and diff to Markdown formatting
│   │   │   ├── index.ts          # Barrel file
│   │   │   └── testing.ts        # Test-only utilities specific to this module
│   │   └── heap-snapshot/        # Common heap snapshot conversion logic
│   │       ├── type.ts           # Parsed heap snapshot types
│   │       ├── graph.ts          # Node adjacency graph in CSR format

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TomerAberbach/profiler-md](https://github.com/TomerAberbach/profiler-md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
