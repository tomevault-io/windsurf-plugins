---
trigger: always_on
description: Fully reconstruct buildable source code from target binaries using the Ralph Loop technique:
---

# Decon Loop — Binary Decompilation via Ralph Loop

## Mission

Fully reconstruct buildable source code from target binaries using the Ralph Loop technique:

1. **Decompile** — Extract structure, logic, and protocols from the binary
2. **Deobfuscate** — If recovered code is obfuscated (mangled names, control flow flattening, string encryption, opaque predicates), reverse the obfuscation to produce human-readable source
3. **Rebuild** — Produce compilable source that can be built back into a functionally equivalent binary

The end goal is a complete, readable, buildable codebase — not just analysis artifacts.

## Design Principle: GENERIC TOOL, NOT TARGET-SPECIFIC

This project builds a **universal binary decompilation pipeline** — it must work on ANY binary, not just the current test target. All prompts, scripts, detection logic, and extraction code must be:

- **Target-agnostic** — no hardcoded variable names, function signatures, or string patterns from a specific binary
- **Format-generic** — handle Mach-O, ELF, PE; native binaries and bundled-app binaries (bun, pkg, nexe, etc.)
- **Prompt-generic** — never include app-specific examples (like OAuth URLs or function names from a particular app) in prompts sent to AI agents

If you find yourself writing code that only works for one specific binary, STOP and make it generic.

## Critical Constraints

- NEVER modify original binaries in `target-binaries/`
- Perform exactly one analysis task per iteration
- Always record discovered code patterns and structural insights in progress.txt
- Save large outputs to files; only write summaries in progress.txt
- Reconstructed source MUST compile. If it doesn't compile yet, mark the task as failing in analysis_plan.json and log the build errors in progress.txt
- When deobfuscating, preserve original logic exactly — rename symbols to meaningful names but never alter behavior

## Target Binaries

All binaries to analyze are in `target-binaries/`. The loop script passes the specific target file via the prompt.

## Multi-Cycle Workflow

The loop repeats **Plan → Build → Verify** until source compiles AND passes quality gates:

### Phase 0: Ghidra Pre-Analysis (once)
- Quick: function boundaries + call graph → `output/ghidra/function_boundaries.tsv`, `call_graph.tsv`
- Full: decompile all functions → `output/ghidra/all_decompiled.c`, `output/ghidra/functions/`
- Module chunks → `output/ghidra/module_chunks.tsv`

### Phase 0.5: Source Discovery & Mapping (once, automatic)
The harness automatically tries to identify the binary and find its source:
1. **`discover_source.py`** — analyzes strings, symbols, embedded paths to identify the software
   - Outputs `output/discovery/identity.json` (name, version, repo URL, languages, confidence)
   - Maintains a known-software database (Bun, Node, Deno, Redis, nginx, etc.) but also does generic detection
   - If identified: shallow-clones the source to `reference-src/<name>/`
2. **`map_to_source.py`** — maps Ghidra functions to original source locations
   - Direct symbol matching (named functions → source function index)
   - String-anchored matching (embedded file paths, error messages → source grep)
   - Call graph propagation (known functions' callees → source callees)
   - Outputs `output/mapping/function_map.tsv`, `helper_aliases.tsv`, `stats.json`

If discovery fails (closed-source binary), the pipeline falls back to pure Ghidra lifting.

### Phase 0.75: Composition Analysis (automatic, when mapping exists)
When source mapping is available (>10% of functions mapped), the pipeline automatically:
1. Categorizes ALL functions: framework (mapped), third-party libraries (by name patterns + call-graph propagation), compiler/system, custom/unknown
2. Identifies bundled third-party libraries (boringssl, zlib, mimalloc, libuv, sqlite, etc.)
3. Outputs `output/composition/analysis.json` with breakdown
4. Auto-selects mode:
   - **`custom_extraction`**: Framework covers >10% → skip known code, focus on custom logic
   - **`full_reconstruction`**: Low mapping → reconstruct everything from Ghidra

### Phase 1: Plan
**Full reconstruction mode** (default):
- If mapping exists: group tasks by SOURCE FILE, prioritize high-confidence mappings
- If no mapping: group by address proximity + call graph clusters
- Every task: `ghidraFunctions`, `addressRange`, `targetSourceFile`, optionally `sourceFiles`
- Output file extension matches source language (.zig, .cpp, .c, .rs)

**Custom extraction mode** (auto-selected):
- Skip framework functions entirely (source already available)
- Focus on clusters of unmapped FUN_ functions using call-graph analysis
- Tasks identify what custom code does and which framework APIs it calls
- Coverage measured against custom function count, not total

### Phase 2: Build (hybrid or pure lifting)
**Hybrid mode** (when reference source is available, full reconstruction):
- Read Ghidra pseudocode AND original source for each mapped function
- Produce output matching the original: same language, names, types, idioms
- Unmapped functions fall back to cleaned Ghidra pseudocode

**Custom extraction mode**:
- Read Ghidra pseudocode for custom function clusters
- Resolve framework API calls using function_map.tsv and helper_aliases.tsv

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tapesymbolstate/decon-loop](https://github.com/tapesymbolstate/decon-loop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
