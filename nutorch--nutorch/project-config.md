---
trigger: always_on
description: **Nutorch v2 (nutorchd) is a shell-agnostic tensor daemon**: GPU-accelerated
---

# Nutorch

**Nutorch v2 (nutorchd) is a shell-agnostic tensor daemon**: GPU-accelerated
PyTorch tensor operations from any shell, built on tch-rs (Rust bindings for
LibTorch, PyTorch's C++ backend).

[Agent development guide](https://agents.md/). `CLAUDE.md` is a symlink to this
file — Claude, Codex, and any other agent read the same contract.

## Rules

Do exactly what your user says. No more, no less. NEVER assume they want
something they didn't ask for. NEVER change code unless explicitly asked.

When editing Rust code, always run `cargo fmt`. Accept the formatter output as
the source of truth. Do not manually undo, minimize, or selectively revert
`cargo fmt` formatting changes, including import ordering or wrapping changes.

Markdown, TOML, and JSON files are formatted with dprint (`dprint fmt`, config
in `dprint.json`). Accept the formatter output as the source of truth.

`v1/` is the archived v1 implementation — a frozen historical reference. Do not
develop in it. The only allowed class of change there is mechanical (e.g. a path
fix required by repo tooling). v2 work ports from v1; it never edits it.

## Vision

v1 proved the core idea: tensors live in a Rust-owned registry, and the shell
passes **string handles** (UUIDs) through pipelines — actual tensor data never
crosses the process boundary. But v1 was a Nushell plugin, so the registry's
lifetime was at the mercy of Nushell's plugin garbage collector, and the
audience was limited to Nushell users.

**v2 decouples the registry from the shell:**

```
bash / zsh / fish / nushell / python / anything
    ↓ thin `torch` CLI client
    ↓ Unix socket (request/response protocol)
nutorchd            ← owns the tensor registry, LibTorch context,
    ↓ tch-rs           GPU memory, and autograd graphs
LibTorch (C++)
    ↓ Metal (MPS)
Apple-silicon GPU
```

- **nutorchd** is a standalone daemon (one copy, or a configurable number) that
  maintains the tensor database. Tensors are referenced by string identifiers.
- **GPU-only, Mac-only for now** (issue 0003): every tensor lives on the GPU —
  on Apple silicon, MPS — and there is **no device option anywhere** in the API.
  The daemon requires MPS and refuses to start without it. Future platform
  expansion (e.g. CUDA on Linux) is a daemon-level "the GPU" decision per
  platform, never a per-tensor option.
- A **thin CLI client** (`torch`) sends one operation per invocation over a Unix
  socket and prints the resulting handle to stdout. Because handles are plain
  text on stdout/stdin, real POSIX pipelines compose — the dual input pattern
  survives in bash almost untouched.
- **Any shell works out of the box.** Nushell remains the premium client
  (structured data, native serialization), but bash, zsh, fish, and scripts in
  any language are first-class citizens.
- **The daemon lifecycle is invisible plumbing** (issue 0004): any `torch`
  command auto-starts the daemon; it shuts itself down after a sliding idle TTL
  (default 1 hour; every tensor op renews the lease), cleaning up its socket on
  every exit path; `torch daemon status|ttl|stop|restart|start` makes it
  analyzable and controllable. Tensors live exactly as long as the daemon — the
  memory-horizon contract. Tensor-level lifecycle (named handles, `free`,
  per-tensor TTLs) remains future work.

The v2 architecture, wire protocol, lifecycle model, and client surface are
designed through issues in `issues/` — the design record lives there, not here.
This section stays a stable summary.

## Carried-Forward Principles (from v1)

v1's architecture record is `v1/AGENTS.md`; its code is the reference
implementation that v2 ports from. These v1 principles remain binding for v2:

1. **String handles are the interface.** Tensor data never leaves Rust; clients
   hold and pass opaque string identifiers.
2. **Dual Input Pattern.** Every operation supports both pipeline form
   (`$t1 | torch add $t2`) and argument form (`torch add $t1 $t2`). This is not
   optional — it is how the tool feels native to both PyTorch users and shell
   users. (v1's "XOR enforcement" clause was retired by issue 0005 in favor of
   the stdin-prefix grammar: stdin fills the leftmost missing tensor slots, one
   handle per line, and is never read when nothing is missing — reading stdin to
   detect a "conflict" blocks on terminals, steals input from enclosing
   `while read` loops, and behaves differently inside pipelines.)
3. **PyTorch API fidelity.** Command names, argument order, defaults, and
   semantics match PyTorch wherever possible.
4. **Explicit over implicit.** No silent auto-casting. (Two clauses retired:
   "manual device placement" by issue 0003 — exactly one device, nothing to
   place — and "no automatic broadcasting" by issue 0005: PyTorch broadcasting
   IS the pledged semantics, and an `add` that disagrees with every PyTorch doc
   would be the real surprise. Non-broadcastable shapes error with both shapes
   named.)
5. **Validate in Rust, not C++.** Pre-validate shapes, dims, and dtypes before
   tch-rs calls — LibTorch errors are opaque and crash-prone; Rust-side
   validation gives good error messages.

## Directory Structure

```
nutorch/
├── README.md                    # Project overview (v2 direction, status)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nutorch/nutorch](https://github.com/nutorch/nutorch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
