---
trigger: always_on
description: Subagent delegation rules — when to delegate static analysis vs run livetools directly, parallel work patterns
---


# Subagent Workflow

The main agent orchestrates and focuses on **live tools**, **dx9tracer capture**, **user interaction**, and **synthesis**. Heavy static analysis and web research are delegated to subagents so the user isn't blocked.

## Pre-flight: Ensure Ghidra Backend

Before first use of pyghidra, the `static-analyzer` subagent should check if Ghidra is available. Run `python verify_install.py` — if pyghidra/Ghidra shows WARN, run `python verify_install.py --setup` to auto-download JDK 21 + Ghidra 11.4.3 + pyghidra. This is a one-time setup (~600MB download). Skip if pyghidra already shows PASS.

## Bootstrap First — New Binaries

When analyzing a binary for the first time (no existing or sparsely populated `patches/<project>/kb.h`), **always bootstrap before other static analysis**:

1. The `static-analyzer` subagent auto-pulls `signatures.db` if missing (pre-flight check). Spawn it to run `bootstrap.py <binary> --project <Name>` — this seeds `patches/<project>/kb.h` with RTTI classes, CRT/library function IDs, compiler info, and propagated labels. **Bootstrap takes 2-5 minutes.** Tell the user it's running and do other work while it completes. The output goes to `patches/<project>/kb.h` — verify this file exists and has content after bootstrap returns. **Bootstrap speeds up all subsequent decompilation**: when `--types kb.h` is passed to the decompiler, it pre-analyzes every known function (`af` per KB entry) so cross-references resolve to named functions, callees get inlined signatures, and you avoid the expensive full-binary `aaa` analysis pass.
2. **In parallel**, spawn a second `static-analyzer` to run `pyghidra_backend.py analyze <binary> --project patches/<Name>`. This runs Ghidra's full analysis (PE loader, MSVC calling convention detection, type propagation, RTTI parsing) and saves a reusable project. **Takes 5-15 minutes.** Once complete, all subsequent decompilations via `--backend auto --project patches/<Name>` will use Ghidra's higher-quality output.
3. Any other static analysis subagents should run in parallel, but their decompilation output will be richer if bootstrap finishes first
4. After bootstrap, all subsequent `decompiler.py` calls **must** use `--types patches/<project>/kb.h`
5. After pyghidra analyze, all subsequent `decompiler.py` calls should also use `--project patches/<project>` so `--backend auto` prefers Ghidra when available

**How to detect "needs bootstrap":** Check if `patches/<project>/kb.h` exists AND has real content (function signatures `@`, globals `$`, or struct definitions beyond section headers). An empty or stub KB with only comment headers counts as sparse — bootstrap it. Quick check: `grep -cE '^[@$]|^struct |^enum ' patches/<project>/kb.h` — if the count is under 50, bootstrap.

**How to detect "needs pyghidra analyze":** Check if `patches/<project>/ghidra/<binary_stem>.gpr` exists. If not, spawn `pyghidra_backend.py analyze`. If kb.h also needs bootstrap, spawn both in parallel.

## Delegation Rules

| Task | Where |
|------|-------|
| Static analysis (`retools`: decompiler, disasm, xrefs, search, structrefs, callgraph, rtti, datarefs, dumpinfo, throwmap) | `static-analyzer` subagent |
| Web research (docs, API refs, format specs, SDK docs) | `web-researcher` subagent |
| Live tools (`livetools`: attach, trace, bp, memwatch, dipcnt, mem read/write) | Main agent — directly |
| dx9tracer trigger/capture | Main agent — directly |
| dx9tracer analyze (offline JSONL analysis) | `static-analyzer` subagent |
| Bootstrap new binary (`bootstrap.py`) | `static-analyzer` subagent -- takes 2-5 min |
| pyghidra analyze (first-time Ghidra analysis) | `static-analyzer` subagent -- takes 5-15 min |
| Decompiler with `--backend ghidra` (subsequent) | `static-analyzer` subagent -- fast (JVM ~3s + decompile <1s) |
| Bulk signature scan (`sigdb.py scan`) | `static-analyzer` subagent -- takes 1-3 min |
| Signature DB build (`sigdb.py build`) | `static-analyzer` subagent -- takes 1-5 min |
| Single function ID (`sigdb.py identify`, `fingerprint`) | Main agent -- fast (<5s) |
| Context assembly (`context.py assemble`) | Main agent -- fast (<5s) |
| Decompiler postprocess (`context.py postprocess`) | Main agent -- instant |
| Dataflow: constants + backward slice (`dataflow.py`) | Main agent -- fast (<5s) |
| File editing, patch specs, builds | Main agent — directly |
| KB updates from subagent findings | `static-analyzer` writes to `kb.h`; main agent may refine |

## Subagent Output Files

Subagents write detailed findings to `patches/<project>/findings.md` (appended, not overwritten). When a subagent returns, it states the file path — **read the file** for full details including decompilation output, address tables, and suggested livetools commands. The return message is just a summary.

## Parallel Work

When both static and dynamic analysis are needed:
1. Spawn `static-analyzer` **in background** for the static questions
2. **Immediately ask the user** if the game/process is running or ask them to launch it — don't wait for static results
3. While the subagent works, prepare livetools (attach, set up traces) or discuss the approach with the user
4. Synthesize findings when the subagent returns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ekozmaster/Vibe-Reverse-Engineering](https://github.com/Ekozmaster/Vibe-Reverse-Engineering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
