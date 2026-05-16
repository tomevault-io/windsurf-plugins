---
trigger: always_on
description: provides better error messages.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Project Overview

**Nutorch is a proof-of-concept Nushell plugin** that demonstrates how to bring
PyTorch tensor operations to the command line by wrapping tch-rs (Rust bindings
for LibTorch, PyTorch's C++ backend).

**This is NOT production-ready software.** It is an experimental project
exploring the intersection of shell scripting and deep learning, with the goal
of making GPU-accelerated tensor operations available directly in the terminal.

### Project Status & Quality Tracking

**See [TODO.md](TODO.md) for complete implementation status and quality
metrics.**

The TODO.md file tracks:

- **Implementation completeness**: Which PyTorch methods are implemented
  (39/200+)
- **Quality criteria**: Test coverage, helper usage, validation, etc. for each
  method
- **Progress to 1.0**: What needs to be done for each method to be
  production-ready
- **Future roadmap**: PyTorch methods not yet implemented

When working on this project, **always consult TODO.md** to:

1. Check which methods need quality improvements
2. See what tests are missing
3. Understand which patterns need to be applied consistently
4. Track overall project completeness

### The Wrapping Layers

```
Nushell (Shell)
    ↓ nu-plugin protocol (MsgPack)
Nutorch Plugin (Rust)
    ↓ tch-rs bindings
LibTorch (C++)
    ↓ native CUDA/MPS/CPU
Hardware (GPU/CPU)
```

Each layer adds abstraction:

- **Nushell**: Shell with structured data (tables, lists, records)
- **Nutorch**: Plugin converting between Nushell values and tensors
- **tch-rs**: Safe Rust API over unsafe C++ FFI
- **LibTorch**: PyTorch's production C++ library
- **Hardware**: Actual compute devices

## Counter-Intuitive Facts

### 1. **Tensors Never Leave Rust**

Despite Nushell being a data-processing shell, **actual tensor data never
crosses the plugin boundary**. Commands pass UUID strings (like
`"a3f2e8c1-..."`) through pipelines, not tensors.

```nushell
# This looks like it passes a tensor through the pipe
let $x = [1 2 3] | torch tensor

# But $x is actually just a string UUID
$x | describe  # → "string"
```

**Why?** Nushell can only serialize simple types (strings, numbers, lists,
records) via MsgPack. Multi-dimensional arrays with GPU backing don't fit this
model.

### 2. **The Registry is a Memory Leak Risk**

The `TENSOR_REGISTRY` is a global HashMap that never automatically clears
(except via garbage collection). Every tensor operation creates a new UUID
entry:

```nushell
# Each intermediate result creates a registry entry
torch full [1000 1000] 1 | torch mm (torch full [1000 1000] 1) | torch mean
# → 3 tensors stored in registry, only the final ID returned
```

**Mitigation**: Nushell's plugin garbage collection (default: 10 seconds) kills
the plugin process, clearing the registry. Users should configure longer GC
intervals:

```nushell
$env.config.plugin_gc = {
  plugins: {
    nutorch: {
      stop_after: 10min  # Prevent premature tensor deletion
    }
  }
}
```

### 3. **In-Place Operations Aren't Really In-Place**

PyTorch has in-place operations (e.g., `tensor.add_(1)`), but Nutorch can't
expose them idiomatically because:

- Nushell is immutable-by-default
- UUID strings are the interface, not tensor references
- `torch sgd_step` appears to mutate, but actually does internal in-place
  updates while returning the same UUID

```nushell
let $w = torch full [2 2] 1 --requires_grad true
# This returns the same UUID, but modifies the underlying tensor
[$w] | torch sgd_step --lr 0.1
```

**Why it works**: The UUID still points to the same PyTorch tensor object, which
was mutated via `f_sub_()` in Rust.

### 4. **Device Placement is Critical and Manual**

Unlike PyTorch which can auto-transfer tensors, Nutorch **requires explicit
device matching**:

```nushell
# This FAILS - device mismatch
let $cpu_tensor = torch full [2 2] 1
let $gpu_tensor = torch full [2 2] 1 --device mps
$cpu_tensor | torch add $gpu_tensor  # ❌ Error from LibTorch

# Must manually ensure both tensors on same device
let $a = torch full [2 2] 1 --device mps
let $b = torch full [2 2] 1 --device mps
$a | torch add $b  # ✅ Works
```

### 5. **Gradients are Stored in PyTorch's Graph, Not Registry**

The `torch grad` command returns a _new_ UUID pointing to the gradient tensor,
but the gradient itself lives in PyTorch's autograd graph attached to the
original tensor.

```nushell
let $w = torch full [1] 2 --requires_grad true
let $loss = $w | torch mean
$loss | torch backward

# This creates a NEW registry entry for the gradient
let $grad_id = $w | torch grad
# The gradient tensor is cloned from PyTorch's graph into registry
```

### 6. **Binary Name ≠ Package Name**

The Cargo.toml specifies:

```toml
[package]
name = "nutorch" # Crate name

[[bin]]
name = "nu_plugin_torch" # Binary name (Nushell convention)
```

This means:

- `cargo install nutorch` installs the package
- But the binary is `~/.cargo/bin/nu_plugin_torch`
- And Nushell commands use `torch` namespace (not `nutorch`)

### 7. **Tests Use NPM, Not Cargo**

Despite being a Rust project, the test suite uses Nushell's testing framework
via NPM:

```bash
cd cargo/test
pnpm install  # Installs test.nu framework

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [astrohackerlabs/nutorch](https://github.com/astrohackerlabs/nutorch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
