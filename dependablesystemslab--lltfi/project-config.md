---
trigger: always_on
description: LLTFI (Low-Level Tensor Fault Injector) is an LLVM-based fault injection framework supporting C/C++ and ML applications (TensorFlow, PyTorch). It injects faults into LLVM IR. It is built on top of LLFI and is fully backward compatible with it.
---

# LLTFI — Claude Context

## What this project is

LLTFI (Low-Level Tensor Fault Injector) is an LLVM-based fault injection framework supporting C/C++ and ML applications (TensorFlow, PyTorch). It injects faults into LLVM IR. It is built on top of LLFI and is fully backward compatible with it.

---

## Build setup (this machine)

| Variable | Path |
|----------|------|
| Source tree | `/home/karthik/Programs/LLTFI` |
| Build root | `/home/karthik/Programs/LLTFI-build` |
| LLVM DST root | `/usr/lib/llvm-20` (apt install) |
| LLVM SRC root | `/home/karthik/Programs/llvm-project` |
| LLVM version | 20.1 |

To rebuild from source:
```bash
./setup -LLFI_BUILD_ROOT /home/karthik/Programs/LLTFI-build \
        -LLVM_SRC_ROOT /home/karthik/Programs/llvm-project \
        -LLVM_DST_ROOT /usr/lib/llvm-20 \
        -LLVM_GXX_BIN_DIR /usr/lib/llvm-20/bin
```

The build root must not already exist. Delete it first if rebuilding from scratch.

To rebuild after code changes (faster):
```bash
cd /home/karthik/Programs/LLTFI-build && make
```

---

## Running the test suite

From the **build** directory:
```bash
cd /home/karthik/Programs/LLTFI-build/test_suite
python3 SCRIPTS/llfi_test --all_cpp                  # all 21 tests
python3 SCRIPTS/llfi_test --all_hardware_faults      # 8 tests
python3 SCRIPTS/llfi_test --all_trace_tools_tests    # 3 tests
python3 SCRIPTS/llfi_test --all_makefile_generation  # 2 tests
python3 SCRIPTS/llfi_test --all_ml                   # ML/ONNX tools
```

Expected: **21/21 PASS** for `--all`. Some error messages during fault injection runs are normal.

`--all_ml` runs additional tests not included in `--all`:

| Test group | Tests | Requirements |
|------------|-------|--------------|
| `CompareLayerOutputs` | 2 | `pip install onnx pygraphviz` |
| `ExtendONNXModel` | 1 | `pip install onnx` |
| `outputONNXGraph` | 1 | `pip install onnx pydot` |
| TensorFlow → ONNX | 3 | `pip install tensorflow tf2onnx onnx` |
| PyTorch → ONNX | 2 | `pip install torch onnx` |
| ONNX → LLVM IR | 2 | `onnx-mlir` + `mlir-translate` on PATH or `$ONNX_MLIR_BUILD` set |
| Fault injection (ML) | 3 | LLTFI build + `model.ll` in `sample_programs/.../mnist/` (run `compile.sh` first) |

Tests with missing deps are reported as **SKIP** (not FAIL) and excluded from the pass/fail count.

The ONNX→IR and fault injection tests use the pre-built `model.onnx` from
`sample_programs/ml_sample_programs/vision_models/mnist/`. The fault injection
test also requires `model.ll` which is produced by that directory's `compile.sh`.

---

## Key architecture

```
llvm_passes/          LLVM pass plugins (compiled to llfi-passes.so)
  core/               Fault injection, profiling, tracing passes
  hardware_failures/  Built-in hardware fault selectors (bitflip, funcname, etc.)
runtime_lib/          Runtime library (libllfi-rt.so) linked into instrumented binaries
bin/                  Python driver scripts: instrument.py, profile.py, injectfault.py
docs/                 input_masterlist.yaml, input_masterlist_ml.yaml — reference schemas
                      for the input.yaml files that control instrumentation and injection
                      input_yaml_guide.md — prose guide to writing input.yaml (user-facing)
tools/                Trace analysis tools (tracediff.py, traceontograph.py, traceunion.py,
                      tracetodot.py), GenerateMakefile/
test_suite/           Regression tests
  PROGRAMS/           Source programs used by tests
  HardwareFaults/     Hardware fault injection test cases
  Traces/             Pre-committed trace reference files for trace tool tests
  MakefileGeneration/ Makefile generation test cases
```

---

## LLVM 19 API constraints

The codebase targets **LLVM 19**. Key API changes to keep in mind:

- `#include "llvm/IR/CFG.h"` (not `llvm/Support/CFG.h` — removed in LLVM 15)
- `CI->arg_size()` (not `CI->getNumArgOperands()` — removed in LLVM 15)
- `func->getName().str()` returns `StringRef`; call `.str()` before assigning to `std::string`
- `opt -load-pass-plugin` (not `opt -load` — legacy PM removed in LLVM 17)
- `--passes=passname` (not `-passname` — old opt syntax removed in LLVM 17)
- `InsertPosition` requires a `BasicBlock::iterator`, not a raw `Instruction*` — call `.getIterator()` on the insertion point
- `getFirstNonPHIOrDbgOrLifetime()` now returns `BasicBlock::iterator` (not `Instruction*`)
- `getFirstNonPHI()` → `getFirstNonPHIIt()` (returns iterator)
- `M.getGlobalList()` is private — use `new GlobalVariable(M, type, ...)` to insert directly
- `itaniumDemangle(str)` takes a single `string_view` argument (old 4-arg form removed)
- All passes use the **new pass manager** (`PassInfoMixin`, `llvmGetPassPluginInfo`); `InstructionDuplication` exposes both a legacy PM class and a new PM wrapper (`NewInstructionDuplicationPass`) registered as `"InstructionDuplicationPass"` in `SEDPasses.so`

---

## Test suite — untracked files that should not be committed

The following files appear as untracked after running tests and should not be staged:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DependableSystemsLab/LLTFI](https://github.com/DependableSystemsLab/LLTFI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
