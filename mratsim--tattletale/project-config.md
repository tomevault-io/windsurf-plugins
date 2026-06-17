---
trigger: always_on
description: Generates: `{.inline.} convertLibTorchExceptions: wrapTorchTensor: F.myNewOp(a.raw, b.raw)`
---

# Tattletale Agent Guidelines

AI coding tool guidelines for Tattletale repo.

## Project

AI inference library in Nim (C++ backend) wrapping libtorch. Tensor ops, safetensors loading, tokenizers, transformer layers.

Python only for test-vector generation (`uv run`). Dev: prefer PyTorch from `uv` / `.venv` (Python 3.14 per `pyproject.toml`) for reference vectors, not system Python.

## Build / Test / Lint

### Dependencies (from `config.nims`)

```bash
nim install_deps          # runtime: nimpy, jsony, stew, packedjson, iface
nim install_deps_dev      # dev: zip (vendor libtorch), chronos (download test tokenizers)
```

### Tests

```bash
nim test_libtorch
nim test_safetensors
nim test_transformers
nim test_toktoktok
```

Single file:
```bash
nim cpp -r --verbosity:0 --hints:off --warnings:off \
  --outdir:build/tests/test_name --nimcache:nimcache/tests/test_name \
  workspace/path/to/test_file.nim
```

### Vendoring

```bash
nim install_libtorch          # download libtorch (needs zip)
nim download_test_tokenizers  # gpt-2 + llama3 fixtures (needs chronos)
nim make_pytoktoktok          # build pytoktoktok.so for Python import
```

### Python

Always `uv run`. `.venv` managed by `uv`:

```bash
uv run --group test-vectors python workspace/module/tests/testgen/generate_vectors.py
```

### CUDA Tests

If working withing this dir, the following might not be needed as
`tattletale/workspace/libtorch/vendor/libtorch.nim` is configuring rpath

To run tests on CUDA, use `LD_PRELOAD` to inject `libtorch_cuda.so` at runtime:

```bash
# Build normally (no -d:cuda needed)
nim cpp -r --hints:off --warnings:off \
  --outdir:build/wip --nimcache:nimcache/wip \
  workspace/transformers/tests/q_exl3/test_*.nim

# Run with CUDA injected
LD_PRELOAD="$(pwd)/.venv/lib/python3.14/site-packages/torch/lib/libtorch_cuda.so" \
LD_LIBRARY_PATH="$(pwd)/.venv/lib:$(pwd)/.venv/lib/python3.14/site-packages/torch/lib:$(pwd)/.venv/lib/python3.14/site-packages/nvidia/cu13/lib" \
build/wip/test_*.nim
```

Test code must move tensors to CUDA via `.cuda()` and `kCUDA` device.

### Code Analysis

`nim cpp` only. `nim check` in C mode lies about C++ exceptions. Hard error on libtorch imports:
> "You are running 'nim check' in C mode. It will misreport that C++ exceptions can't be caught because they aren't ref objects."

## Architecture: Tensor Layer

### `Tensor` — Ref Wrapper

`TorchTensor` (raw `importcpp: "torch::Tensor"`) never public. Wrapped in `ref object` `Tensor` (`tensors.nim:29`):

```nim
type Tensor* = ref object
  raw: TorchTensor
```

Avoids Nim C++ FFI issues with value types in containers (copy-ctor, `=wasMoved`, `{}` default init).

**Contract:** No C++ types (`TorchTensor`, `CppVector`, `CppTuple`, `IntArrayRef`, `ArrayRef`, `CppString`) leak past `tensors.nim` / `tensors_nn.nim`. `workspace/libtorch` (`libtorch.nim:1-2`) only re-exports those two.

### `wrapLibtorch:` Macro — Auto-Forwarding

`tensors.nim:177`. Inside `wrapLibtorch:` block, every `proc`/`func` gets:

1. `{.inline.}` auto-added
2. C++ exceptions → `LibTorchDefect` via `convertLibTorchExceptions` (`tensors.nim:71`)
3. Return wrapped via `wrapTorchTensor` (`tensors.nim:43`) — no-op for non-Tensor returns
4. Inputs unwrapped via `unwrapArg` (`tensors.nim:121`) — `Tensor` → `.raw`, `varargs[int]` → `asTorchView`, `typedesc` → `toScalarKind`

**Bare signatures** (~95% of procs). `autoForward` (`tensors.nim:138`) generates forwarding body:

```nim
wrapLibtorch:
  func zeros*(size: varargs[int]): Tensor
  func zeros*(size: varargs[int], T: typedesc[SomeTorchType]): Tensor
  func dim*(a: Tensor): int
```

`autoForward` generates: `{.inline.} convertLibTorchExceptions: wrapTorchTensor: F.zeros(asTorchView(size))`

**Explicit bodies** bypass auto-forward. Write wrapping manually (`tensors.nim:505`):

```nim
func `<.`*(a: Tensor, b: Tensor): Tensor {.inline.} =
  convertLibTorchExceptions:
    wrapTorchTensorImpl:
      F.lt(a.raw, b.raw)
```

Templates always explicit (`tensors.nim:392`):

```nim
template sizes*(a: Tensor): openArray[int] =
  F.sizes(a.raw).asNimView()
```

**`wrapTorchTensor`** (`tensors.nim:43`): `when typeof(...)` dispatch — `TorchTensor` → `Tensor`, `CppVector[TorchTensor]` → `seq[Tensor]`, `CppTuple2` → tuple. No-op else.

### C++ Exception Handling

`LibTorchDefect` inherits `Defect` (`tensors.nim:69`). Keeps `raises: []` clean.

`convertLibTorchExceptions` (`tensors.nim:71`) catches `TorchError` (`c10::Error`), re-raises as `LibTorchDefect` with `.what()` message.

`LibTorchDefect` exported — test harnesses can `except LibTorchDefect`.

### Module Structure (libtorch)

```
workspace/libtorch/libtorch.nim        — re-export: tensors + tensors_nn [line 1-2]

PUBLIC (no C++ types leak):
  src/tensors.nim          — Core wrapper (Tensor + ~500 procs)
  src/tensors_nn.nim       — NN functional (activations, losses, SDPA)
  src/tensors_py.nim       — Nim ↔ Python Tensor bridge

INTERNAL (not exported):
  src/raw/abi/torch_tensors.nim   — Pure C++ FFI (~265 procs)
  src/raw/abi/neural_nets.nim     — C++ FFI neural net API
  src/raw/abi/c10.nim             — TorchError, IntArrayRef
  src/raw/abi/std_cpp.nim         — CppStdException, what()
  src/raw/torch_tensors_sugar.nim — asTorchView, indexing macros

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mratsim/tattletale](https://github.com/mratsim/tattletale) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
