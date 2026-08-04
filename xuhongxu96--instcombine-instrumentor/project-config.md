---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo does

This is *not* a normal codebase — it is a build harness that patches an upstream LLVM source tree and produces an instrumented `opt` binary that traces every new instruction and every RAUW performed by `InstCombine` / `InstructionSimplify` per pass iteration. The first-party sources are `patch_llvm.py` (the patcher), the C++ runtime under `runtime/`, the minimal wasm driver under `wasm/driver/`, the webapp under `web/`, plus a handful of shell scripts. Live webapp: <https://xuhongxu.com/instcombine-instrumentor/>.

The C++ runtime injected into LLVM lives in real source files at `runtime/fuzz_runtime.{h,cpp}` — edit those, not `patch_llvm.py`. The patcher reads them at module import via `Path.read_text()` and writes them into the LLVM tree at `llvm/include/llvm/IR/fuzz_runtime.h` and `llvm/lib/IR/fuzz_runtime.cpp`. The runtime is target-agnostic for the trace path — call-path frames come from a self-maintained `thread_local` stack populated by an RAII `CallScope` pushed *at each call site*, not from `llvm::sys::PrintStackTrace`, so native and wasm traces are byte-format-identical. The only remaining `#ifdef __EMSCRIPTEN__` guards `std::atexit` (unreliable under emscripten); an always-emitted `extern "C" dump_iteration_info_external` lets the wasm host flush the final iteration explicitly.

## Common commands

```bash
uv sync                                            # install Python deps (tree-sitter)
bash clone_llvm.sh                                 # clone LLVM at the ref in llvm_commit.txt
uv run python patch_llvm.py --llvm-repo thirdparty/llvm-project
bash build_patched_llvm.sh                         # builds opt + llvm-symbolizer into build/llvm-rel
bash smoke_test.sh                                 # runs opt on a tiny IR and checks the trace
```

To bump the LLVM version: edit `llvm_commit.txt`, re-run `clone_llvm.sh` → `patch_llvm.py` → `build_patched_llvm.sh`. `patch_llvm.py` is idempotent on a clean checkout, so running it twice is a no-op (each patcher checks for sentinels like `__llvm_fuzz_record_replace`, `llvm_fuzz::start_iteration()`, `fuzz_runtime.cpp`). It also carries one small build-graph workaround for newer LLVM snapshots: if `llvm/lib/MC/DXContainerInfo.cpp` includes `llvm/Support/VCSRevision.h` but `llvm/lib/MC/CMakeLists.txt` does not depend on `llvm_vcsrevision_h`, the patcher adds that missing dependency so parallel wasm builds do not race on the generated header.

At runtime, `DISABLE_INSTCOMBINE_TRACE=1` makes the patched `opt` behave like stock `opt`. The trace file is hard-coded to `llvm_fuzz_info.txt` in the CWD.

## Patching architecture

`patch_llvm.py` uses tree-sitter (not regex/sed) to walk C++ function definitions. Patching runs in **two passes**:

1. **First pass (`_collect_instrumented_names`)** scans every `.cpp`/`.h` under `llvm/lib/Transforms/InstCombine/` plus `llvm/lib/Analysis/InstructionSimplify.cpp` and collects the bare names of every function whose return type "looks like a pointer to a Value/Instruction subclass" (see `POINTER_TYPE_HINTS` and `_is_pointer_return`). This name set is the **call-site allowlist** used by the second pass.

2. **Second pass** applies three distinct patchers to specific files in `thirdparty/llvm-project`:

   1. **`patch_value_cpp`** → `llvm/lib/IR/Value.cpp`. Finds `doRAUW` and inserts `__llvm_fuzz_record_replace(this, New)` at the top of its body. This is the single RAUW hook for the whole engine.

   2. **`patch_inst_combine_file`** → every `.cpp`/`.h` in `llvm/lib/Transforms/InstCombine/`. For *every* function body (not just pointer-returning ones), it wraps each `call_expression` whose bare callee is in the allowlist or matches `^Create[A-Z]` (IRBuilder `CreateAdd`/`CreateICmp`/… without enumerating dozens of variants) as `__llvm_fuzz_call(callee(args...))`. A call that is the immediate operand of a unary address-of (`&foo()`, walking through parens) is skipped (`_is_address_of_operand`): the `__llvm_fuzz_call` statement-expression yields a temporary rvalue, so `&__llvm_fuzz_call(expr)` won't compile even when `expr` is itself an lvalue (e.g. `&CI->getValue()`). Additionally, for pointer-returning functions it wraps every *top-level* `return X;` as `return __llvm_fuzz_record(static_cast<class T*>(X));` (returns inside nested lambdas/local function definitions are skipped via `is_inside_nested_scope`). The `static_cast<class T*>` — where `T*` is the function's own pointer return type, recovered by `_extract_pointer_return_type` — forces any implicit conversion (e.g. a returned `Use&` that converts to `Value*`) *before* `__llvm_fuzz_record`'s template argument deduction sees the expression, which would otherwise fail to deduce. In `InstructionCombining.cpp` the InstCombine top-level `run` method also gets a `llvm_fuzz::start_iteration()` prelude and its `return MadeIRChange;` is rewritten to dump the iteration trace first; it's matched as either `InstCombinerImpl::run` (LLVM ≥ 6) or `InstCombiner::run` (LLVM ≤ 5, before the class was renamed).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xuhongxu96/instcombine-instrumentor](https://github.com/xuhongxu96/instcombine-instrumentor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
