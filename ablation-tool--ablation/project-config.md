---
trigger: always_on
description: Binary RE toolkit for stripped firmware. No symbols. No source.
---

# Ablation: Claude Operational Reference

## What this tool is

Binary RE toolkit for stripped firmware. No symbols. No source.

`pip install -e ~/ablation/` (editable install, already done)

---

## Session start: run this every time

```python
from ablation.analyzers.binary_context import BinaryContext

# 1. Read SESSION.md (root) + targets/<vendor>/SESSION_<target>.md
# 2. Load context (0.5s first run, 110ms from cache)
ctx = BinaryContext.load_or_build('/path/to/target.so')
print(ctx.summary())

# 3. Surface all confirmed function names before anything else
if ctx.names_count():
    print(ctx.names_table())

# RULE: ctx.name(va) everywhere. Never raw hex in display contexts.
# ctx.set_name(0x17b660, "ips_diameter_parse_message", source="confirmed")
```

SESSION.md convention: root index at `~/ablation/SESSION.md`; per-target state at `targets/<vendor>/SESSION_<target>.md`. Read before touching any binary. Update at end of each session.

---

## Which tool for which task?

| Task | First reach |
|---|---|
| "What does this function call?" | `ctx.callees_of(va)` |
| "What calls this symbol?" | `ctx.callers_of('symbol')` or `ctx.callers_of(va)` |
| "What strings does this function reference?" | `ctx.strings_in_func(va)` |
| "Which functions reference this string?" | `ctx.funcs_referencing_string(string_va)` |
| "What is this function?" | `ctx.name(va)` |
| "Show disassembly around address" | `WindowAnalyzer.dump_text(va, window=1536)` |
| "Find functions matching vulnerability pattern" | `SemanticSearcher.query(description)` (ALWAYS first on new binary) |
| "What values are passed to this sink?" | `FuncProfiler.profile(va).fmt()` |
| "Trace taint from network recv to sink" | `TaintTracker.run_interprocedural()` |
| "ARM32: trace recv to malloc/strcpy/system" | `ARM32TaintTracker.from_context(ctx).run_interprocedural()` |
| "ARM32: find MUL before malloc without bounds check" | `ARM32IntOverflowScanner.from_context(ctx).scan()` |
| "MIPS32: trace recv to system/strcpy/sprintf" | `MIPS32TaintTracker.from_path(elf).run_interprocedural()` |
| "MIPS32: big-endian RouterOS or little-endian CPE" | `MIPS32TaintTracker.from_path(elf, endian='big')` |
| "MIPS64: trace recv (Cisco IOS/OCTEON big-endian)" | `MIPS64TaintTracker.from_path(elf, endian='big').run_interprocedural()` |
| "MIPS64: little-endian RouterOS 64" | `MIPS64TaintTracker.from_path(elf, endian='little').run_interprocedural()` |
| "nanoMIPS: walk frame boundaries" | `NanoMIPSDecoder(endian='little').decode_frames(data, base_addr)` |
| "PPC32: trace recv (Cisco IOS 7200, VxWorks)" | `PPC32TaintTracker.from_path(elf, endian='big').run_interprocedural()` |
| "PPC64: trace recv (IBM POWER, AIX)" | `PPC64TaintTracker.from_path(elf, endian='big').run_interprocedural()` |
| "ARC: trace recv (ARC HS IoT, Marvell, Seagate)" | `ARCTaintTracker.from_path(elf).run_interprocedural()` |
| "RISC-V 32: trace recv (SiFive, Allwinner D1)" | `RISCV32TaintTracker.from_path(elf).run_interprocedural()` |
| "RISC-V 64: trace recv (VisionFive 2, SiFive Unmatched)" | `RISCV64TaintTracker.from_path(elf).run_interprocedural()` |
| "V850: trace recv (RH850/G3M ECU)" | `V850TaintTracker.from_path(elf).run_interprocedural()` |
| "Find printf/syslog with non-literal format string" | `FormatStringScanner.from_context(ctx).scan()` |
| "Scan for heap integer overflow / UAF / double-free" | `HeapVulnScanner.from_context(ctx).scan()` |
| "Where did this command string come from? (system/popen/execve)" | `SinkArgClassifier.from_path(elf).classify_all()` — verdicts: RODATA_CONST/SNPRINTF_RODATA/ARG_PROPAGATED/UNKNOWN |
| "Register vendor-specific sinks + known-safe patterns" | `VendorProfile.from_vendor('fortinet').apply_to(clf)` — loads profiles/fortinet.yaml; zero-caller sinks auto-ELIMINATED |
| "Which libs in a rootfs dir have exec-class PLT imports?" | `batch_plt_intersect('/tmp/fad_root/lib/')` — returns {path:[sinks]}; omitted=auto-CLEAN; run FIRST, audit only the hits |
| "Detect allowlist byte-validators in stripped binary" | `SanitizerDetector.from_path(elf).detect()` — SHELL_SAFE/SHELL_UNSAFE/UNKNOWN per charset |
| "Classify fork() callers as worker/exec/exit" | `ForkExecClassifier.from_path(elf).classify()` — WORKER/EXEC_AFTER_FORK/EXIT_IN_CHILD |
| "Find 'safe now catastrophic later' rendering architecture risk (TS/JS/Python)" | `SourceArchRiskScanner.from_context(ctx).scan()` — tags: unsafe_render/type_dispatch/string_selector/registry_lookup/shared_module; HIGH=score≥3 or known combo |
| "Compress N-file source audit to M profile buckets (40x read reduction)" | `SourceAuditCompressor.from_context(ctx).compress()` — 5-bit profile per file; profile 0=batch-CLEAN; profiles 8-31=individual reads; .compression_ratio() gives % reads saved |
| "Trace an arg across 3 library hops" | `IPRegAnnotator.annotate_chain(va, max_hops=3)` |
| "Which library exports this symbol?" | `LibGraph.defined_in('symbol')` |
| "Is this the same function as in v7.4?" | `DTWMatcher.score_functions(va_a, va_b)` |
| "Where did this function change across versions?" | `MatrixProfileDiff.diff_functions(va_v1, va_v2)` |
| "Confirm taint path is reachable" | `PathSolver.solve_path(func_va, target_va)` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ablation-Tool/ablation](https://github.com/Ablation-Tool/ablation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
