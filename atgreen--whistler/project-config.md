---
trigger: always_on
description: A Lisp that compiles to eBPF. Written in Common Lisp (SBCL).
---

# Whistler

A Lisp that compiles to eBPF. Written in Common Lisp (SBCL).

## Build & Run

```bash
# Load and compile a program
sbcl --noinform --non-interactive \
  --eval '(require :asdf)' \
  --eval '(push #P"/home/green/git/whistler/" asdf:*central-registry*)' \
  --eval '(asdf:load-system "whistler")' \
  --eval '(whistler::compile-file* "examples/synflood-xdp.lisp" "output.bpf.o")'

# Disassemble (inspect instruction output)
# Use whistler::disassemble-cu on the compilation-unit returned by compile-to-elf
```

`compile-file*` and `with-bpf-session` automatically isolate compilation state. When using `compile-to-elf` directly in the REPL, call `(reset-compilation-state)` between separate compilations to clear accumulated maps/programs/structs.

No test suite. Verify correctness by checking instruction counts and comparing disassembly output.

## Architecture

Pipeline: **source** → macro expansion → **lowering** (lower.lisp) → SSA IR → **optimization** (ssa-opt.lisp) → **register allocation** (regalloc.lisp) → **BPF emission** (emit.lisp) → **peephole** (peephole.lisp) → **ELF output** (elf.lisp)

### Key files

| File | Purpose |
|------|---------|
| `packages.lisp` | Package definitions and exports |
| `bpf.lisp` | BPF instruction encoding, constants, opcodes |
| `compiler.lisp` | Legacy direct compiler, macro expansion (`whistler-macroexpand`), constant folding, **shared definitions** (helpers, constants, builtins, context struct layouts, BTF resolver hook -- single source of truth for both pipelines) |
| `ir.lisp` | SSA IR data structures (`ir-insn`, `basic-block`, `ir-program`) |
| `lower.lisp` | Lowering from surface language to SSA IR |
| `ssa-opt.lisp` | SSA optimizations (copy prop, DCE, constant folding, phi threading) |
| `regalloc.lisp` | Linear-scan register allocator with spilling |
| `emit.lisp` | IR → BPF instruction emission, stack allocation, map operations, tail calls |
| `peephole.lisp` | Post-regalloc BPF peephole optimizer (15+ passes) |
| `btf.lisp` | BTF type encoding and BTF.ext (CO-RE relocations, func_info) |
| `elf.lisp` | ELF object file writer (multi-program support) |
| `protocols.lisp` | Protocol header macros (Ethernet, IPv4, TCP, UDP), map/struct surface macros |
| `vmlinux.lisp` | BTF reader, `import-kernel-struct`, context struct BTF lookup, CO-RE resolver |
| `whistler.lisp` | Top-level interface: `defmap`, `defprog`, `defstruct`, `compile-to-elf` |

### Packages

- `whistler/bpf` — BPF constants and instruction constructors
- `whistler/compiler` — Legacy compiler, macro expansion, **shared definitions** (`*builtin-helpers*`, `*builtin-constants*`, `*whistler-builtins*`, `sym=`, `bpf-type-p`)
- `whistler/ir` — IR, lowering, optimization, regalloc, emission, peephole
- `whistler/elf` — ELF output
- `whistler/btf` — BTF and BTF.ext encoding
- `whistler` — User-facing surface language (programs use `(in-package #:whistler)`)

## Surface Language

Programs are defined with `defprog`, maps with `defmap`, structs with `defstruct`. Standard CL `let` bindings with optional type inference:

```lisp
(let ((x (load u32 ptr 0))       ; type inferred from load → u32
      (y (tcp-flags tcp)))        ; inferred → u8
  (store u32 ptr 4 x))           ; memory store
```

Use `(declare (type ...))` for narrowing when inference can't determine the type (integer literals, arithmetic results).

Key forms: `let` (parallel bindings, standard CL), `let*` (sequential bindings), `if`, `when`, `unless`, `when-let`, `if-let`, `return`, `load`, `store`, `logand`, `logxor`, `>>`, `ash`, `cast`, `ctx`, `map-lookup`, `map-update`, `map-delete`, `map-lookup-ptr`, `struct-alloc`, `stack-addr`, `tail-call`, `get-prandom-u32`, `sizeof`, `memset`, `memcpy`, `do-user-ptrs`, `do-user-array`, `with-ringbuf`, `fill-process-info`, `pt-regs-parm1`..`parm6`, `pt-regs-ret`, protocol accessors.

`setf` supports CL-style multi-pair: `(setf place1 val1 place2 val2 ...)`. `defmap` defaults `:key-size` and `:value-size` to 0 (omit for ringbuf maps).

`defstruct` generates BPF accessors `(name-field ptr)`, `setf` expanders, indexed array access, and pointer accessors `(name-field-ptr ptr)`. Also generates CL-side: `name` struct with `(name-field instance)` accessors, `decode-name` (bytes→struct), `encode-name` (struct→bytes). The CL accessors use the same names as the BPF accessors. `(sizeof name)` returns compile-time struct size. Maps support `(getmap m k)` / `(setf (getmap m k) v)` / `(remmap m k)` — matching CL's gethash/remhash pattern.

`with-ringbuf` handles reserve/null-check/submit: `(with-ringbuf (var map size) body...)`. `fill-process-info` fills pid/uid/timestamp/comm from BPF helpers using struct accessor names.

Context access: `(ctx field-name)` reads a field from the program's context struct, resolved by program type (e.g., `:xdp` uses `xdp_md`, `:cgroup-sock-addr` uses `bpf_sock_addr`). `(setf (ctx field-name) val)` writes. Array fields: `(ctx user-ip6 0)`. Legacy `(ctx u32 4)` with explicit type+offset still works. Field-name access emits CO-RE relocations for compile-once portability; offsets are resolved from BTF at compile time when `/sys/kernel/btf/vmlinux` is available, falling back to a static table.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atgreen/Whistler](https://github.com/atgreen/Whistler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
