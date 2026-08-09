---
trigger: always_on
description: Matching decompilation of LEGO Star Wars: The Video Game (2005) for GameCube.
---

# LSW1 Decomp Agent Guide

Matching decompilation of LEGO Star Wars: The Video Game (2005) for GameCube.
Retail target: `GL5E4F`. Compiler: **SN Systems ProDG (GCC)**, driven by
`tools/prodg_cc.py` — *not* Metrowerks CodeWarrior. See
[docs/code_matching_workflow.md](docs/code_matching_workflow.md). (Donor decomps
like CrashWOC are MWCC, so their source patterns transfer but their codegen does
not.)

## Quick start

```sh
bash build.sh                                    # full build
ninja build/GL5E4F/report.json                   # refresh progress

python tools/ls1_match_plan.py nuanim            # see unnamed candidates
python tools/ls1_lookup_symbol.py 0x8001E76C     # cross-reference
python tools/ls1_rename.py --suggest fn_8001E76C # check Mac for name ideas
python tools/ls1_rename.py fn_8001E76C NuAnimKeyLerp  # apply + verify
python tools/ls1_task_pack.py NuAnimKeyLerp      # generate task pack for C work
python tools/matching_progress.py                # show current code matching progress
python tools/binary_mining_pipeline.py           # refresh donor-binary evidence
bash build.sh                                    # final check
```

## Symbol sources (priority order)

1. **CrashWOC decomp** (same Nu2 engine) — `docs/nu2_engine_reference.md`,
   `docs/symbol_donors/crashwoc_nu2_comparison.md`
2. **LSW1 Mac Demo** (closest lineage) — `docs/symbol_donors/mac_lsw1_demo_symbols.tsv`
3. **LSW2 Mac** (shared Nu2 engine) — `docs/symbol_donors/mac_lsw2_symbols.tsv`
4. **Struct offset analysis** — match struct field access patterns against
   known Nu2 structs in `src/` headers

## Tooling

| Command | What it does |
|---------|-------------|
| `ls1_match_plan.py <module>` | Shows remaining unnamed functions with struct evidence, Mac hints. Use `--show-all` or `--top N` |
| `ls1_lookup_symbol.py <name-or-addr>` | Cross-references a symbol across GC symbols.txt, Mac TSVs (with demangled signatures), call graph, and neighbor functions |
| `ls1_rename.py <old> <new>` | Applies a rename and verifies with build. Use `--check` to preview, `--suggest <fn>` to search Mac for matching names |
| `ls1_task_pack.py <name-or-addr>` | Generates `tasks/<Function>/` with `prompt.md`, `context.md`, `asm.s`, `related_symbols.md`, `verify.sh` for LLM-based decomp. Use `--out-dir` to set output root. Batch: `ls1_task_pack.py nuanim --top 3` |
| `matching_progress.py [--report] [--json]` | Shows objdiff code matching progress only, without symbol naming stats |
| `binary_mining_pipeline.py [stage...]` | Refreshes donor-binary extraction artifacts; see `docs/binary_mining_workflow.md` |
| `sdk_symbol_source_import.py <source...>` | Normalizes CrashWOC/Dolphin map files or symbol dumps into ordered SDK TSVs |
| `mac_anchor_rename_queue.py` | Uses Mac debug symbol order plus GC named anchors to emit rename candidates |
| `sdk_island_analysis.py` | Maps likely Dolphin SDK/MSL/runtime islands for known-library matching |
| `sdk_anchor_rename_queue.py` | Uses local SDK symbol-order sources plus GC named anchors to emit SDK rename candidates |

## Code matching workflow

Use [docs/code_matching_workflow.md](docs/code_matching_workflow.md) as the
source of truth for C matching.

Tool roles:

| Tool | Role |
|------|------|
| `crashwoc_code_queue.py` | Generates donor-guided candidate rows from a local CrashWOC checkout. It does not change build wiring. |
| `verify_fn.py` | Compiles a candidate object and compares its instruction stream against the target asm. This is the primary scoring loop. |
| `asm_function_slice.py` | Plans the before/C/after split for a verified function. `--write` only emits generated slices; it does not automatically wire them into the build. |
| `symbols_to_map.py --all` | Produces the Dolphin `.map` file with `zz_<address>_` placeholders. This is separate from code matching. |
| `matching_progress.py` | Reads objdiff progress only. It does not validate a candidate function. |

## Matching workflow (symbol recovery)

1. `python tools/binary_mining_pipeline.py` — refresh donor evidence and rename queues
2. Review `docs/symbol_donors/mac_anchor_rename_queue.md` for high-confidence candidates
3. `python tools/ls1_match_plan.py <module>` — see remaining module candidates
4. `python tools/ls1_lookup_symbol.py <fn_addr>` — verify struct types in Mac
5. `python tools/ls1_rename.py --suggest <fn_name>` — check Mac for name hints
6. `python tools/ls1_rename.py <old> <new>` — apply and build-verify
7. `bash build.sh` — final check
8. Commit after each module phase

## C matching workflow

1. `python tools/crashwoc_code_queue.py /tmp/crashwoc-decomp` — refresh exact donor-guided candidates
2. `python tools/ls1_task_pack.py <fn_name> --out-dir work` — generate task pack
3. Write matching C in `work/<Fn>/source.c` (or inline into `src/<module>/<file>.c`)
4. `python3 configure.py && ninja build/GL5E4F/src/<unit>.o` — compile the candidate object
5. `python3 tools/verify_fn.py <fn_name> --unit <unit> [--compiled-symbol <symbol>]` — compare compiled instructions against the target
6. When the candidate is proven, add the split and object wiring, then re-run `python3 configure.py && ninja`

## Module address ranges

| Module | Start | End | Est. size |
|--------|-------|-----|-----------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ellierocks/lsw1-decomp](https://github.com/ellierocks/lsw1-decomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-08 -->
