---
trigger: always_on
description: Uniprof is a universal profiling tool that wraps platform profilers (py-spy, 0x, rbspy, async-profiler, dotnet-trace, perf, xctrace, etc.) to provide a consistent, cross‑language experience. It prefers Docker to minimize setup and ensure reproducible results.
---

# Uniprof Codebase Guide

Uniprof is a universal profiling tool that wraps platform profilers (py-spy, 0x, rbspy, async-profiler, dotnet-trace, perf, xctrace, etc.) to provide a consistent, cross‑language experience. It prefers Docker to minimize setup and ensure reproducible results.

## Project Layout

```
uniprof/
├── src/
│   ├── index.ts            # CLI entry
│   ├── version.ts          # Single source of truth for version
│   ├── commands/           # CLI command implementations
│   ├── mcp/                # Model Context Protocol server + tools
│   ├── platforms/          # Language/platform plugins
│   ├── types/              # Type definitions
│   └── utils/              # Shared utilities
├── containers/             # Docker images per platform
├── speedscope/             # Bundled Speedscope viewer
├── tests/                  # Test suite
└── docs/                   # Documentation
```

## Quick Start

- Simplest: profile and analyze in one step
  - `uniprof python app.py`
  - `uniprof --visualize node server.js`
- Save then analyze/visualize later
  - `uniprof record -o profile.json -- ./my-app`
  - `uniprof analyze profile.json`
  - `uniprof visualize profile.json`

## CLI Reference

### Aliases & Parsing
- `uniprof <cmd>` ≈ `uniprof record --analyze -- <cmd>` (unless `--visualize`, which maps to `record --visualize`).
- Options before the first non‑option belong to uniprof; after, they belong to your command.
- `record` inserts `--` automatically when omitted (so trailing args reach your app).
- `--extra-profiler-args` accepts dashed values directly or as a single quoted string; the alias layer condenses them.

Examples
```
uniprof python app.py
uniprof -- python app.py
uniprof --visualize -- node server.js
uniprof -o out.json --analyze -- ./my-app
```

### Commands
- `bootstrap` — Environment checks and setup guidance
- `record` — Record a profile
- `analyze` — Analyze an existing profile
- `visualize` — Serve the profile in a local Speedscope
- `mcp` — Run or install the MCP server

Advanced options
```
uniprof record --mode host -o profile.json -- python app.py
uniprof record -o profile.json --extra-profiler-args --rate 500 -- python app.py
uniprof analyze profile.json --threshold 5 --filter-regex "MyApp\." --min-samples 100 --max-depth 10
```

### Path Validation (Container Mode)
- Absolute argument paths outside the working directory cause errors; they won’t be mounted.
- Option values that look like paths outside the working directory trigger warnings.
- Host mode skips these checks.

## Platforms

### Supported & Defaults
- Sampling defaults (≈999Hz) unless platform‑fixed:
  - Python/Ruby: `--rate 999`
  - PHP: `--period 0.001001001` (≈999 Hz)
  - JVM: `--interval 1001001ns`
  - perf/BEAM: `-F 999`
  - Node.js (0x), xctrace: fixed by runtime/system
- Users can override via `--extra-profiler-args`.

### macOS `.app` Bundles (xctrace)
- Resolves app bundles to CFBundleExecutable under `Contents/MacOS` and validates permissions.

## Containers & Host

- Docker is preferred; host mode is available per platform.
- Host networking: supported on Linux; on macOS requires Docker Desktop host‑networking support (see Docker settings/diagnostics). When requested and not available, a warning is shown.
- Symbol resolution with perf in containers uses `--symfs /` and build‑id caching; some paths may still appear unknown.

## Analysis

### Profile Types
- Sampled: py-spy, rbspy, Excimer, async-profiler (in sampled mode), 0x, perf
  - Statistical samples with sample counts correlated to CPU time
- Evented: dotnet-trace (EventPipe), Instruments
  - Entry/exit events converted to synthetic weighted samples for consistent analysis

### Analyze Options
- `--threshold <percent>`: Minimum percent of total time to display (default 0.1)
- `--filter-regex <pattern>`: Filter by function or file path
- `--min-samples <count>`: Minimum sample count to show
- `--max-depth <depth>`: Consider leaf‑most N frames per sample

Implementation details
- Evented → sampled conversion attributes elapsed time to the current stack; computes totals, self time, and percentiles when weights vary.
- .NET: Uses EventPipe `cpu-sampling` with precise timing and full call stacks; use `--platform dotnet` if auto‑detect fails.

## Visualization

### Speedscope Viewer
- Build copies `speedscope/` to `dist/speedscope`.
- At runtime, serve from `dist/speedscope` or fallback to repo `speedscope/`.
- If not found, run `npm run build`.

## MCP Integration

Commands
```
uniprof mcp run
uniprof mcp install <client>
```

Tool: `run_profiler`
- Required: `command`, `cwd`
- Optional: `platform`, `mode`, `output_path`, `enable_host_networking`, `extra_profiler_args`, `verbose`
- Returns analyzed results (pretty or JSON)

Schema notes
- The MCP tool registers a Zod schema shape (not wrapped in `z.object(...)`). Convert to JSON Schema at client integration boundaries when needed; do not change the shape in `src/mcp/tools.ts`.
- Zod import path is intentionally `zod/v3` in MCP files to align with SDK expectations; do not change this import.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [indragiek/uniprof](https://github.com/indragiek/uniprof) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
