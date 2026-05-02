---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Python 3.12 bytecode interpreter in x86-64 NASM assembly. Reads `.pyc` files and executes bytecode directly.

## Build & Test

```bash
make              # build ./apython
make clean        # remove build/ and apython
make check        # full test suite: compile .py→.pyc, diff python3 vs ./apython output
make check-cpython # CPython stdlib unit tests (harder, more thorough)
```

**Always run BOTH `make check` AND `make check-cpython` to verify changes.**

**Single test:**
```bash
python3 -m py_compile tests/test_foo.py
python3 tests/test_foo.py > /tmp/expected.txt
./apython tests/__pycache__/test_foo.cpython-312.pyc > /tmp/actual.txt
diff /tmp/expected.txt /tmp/actual.txt
```

**Dependencies:** nasm, gcc (linker), libgmp-dev, python3.12

## Register Convention (eval loop)

Callee-saved registers hold global interpreter state:

| Register | Role |
|----------|------|
| `rbx` | Bytecode IP (into co_code[]) |
| `r12` | Current frame (PyFrame*) |
| `r13` | Value stack top (payload array, u64[]) |
| `r14` | co_consts data ptr (&tuple.ob_item[0]) |
| `r15` | Tag stack top (sidecar tag array, u8[]) |
| `ecx` | Opcode arg on handler entry |

co_names is accessed via `LOAD_CO_NAMES reg` / `LOAD_CO_NAMES_TAGS reg` macros (reads from `eval_co_names` / `eval_co_names_tags` globals), not a dedicated register.

**Critical rule:** Never hold live values in caller-saved regs (rax, rcx, rdx, rsi, rdi, r8-r11) across `call` or `DECREF`/`DECREF_REG`. Use push/pop or callee-saved regs instead. `DECREF_REG` calls `obj_dealloc` which clobbers all caller-saved regs.

## Value64 Representation

Values are split into 64-bit payloads stored in `u64[]` arrays and 8-bit tags stored in separate `u8[]` sidecar arrays. The value stack uses `r13` (payload top) and `r15` (tag top). Containers (list, tuple, dict) store `ob_item` (u64[]) and `ob_item_tags` (u8[]) separately. Frame locals use `localsplus` (u64[]) and `locals_tag_base` (u8[]).

Tags (u8): `TAG_NULL=0`, `TAG_SMALLINT=1`, `TAG_FLOAT=2`, `TAG_NONE=3`, `TAG_BOOL=4`, `TAG_PTR=0x85`. Bit 7 (`TAG_RC_BIT=0x80`) means payload is a refcounted heap pointer. SmallInts store raw signed i64 in payload (full 64-bit range), zero heap alloc/refcount. `INCREF_VAL`/`DECREF_VAL` check `TAG_RC_BIT` to decide refcounting. Functions return `(rax=payload, edx=tag)`.

## Source Layout

- `src/eval.asm` — Bytecode dispatch loop (256-entry jump table)
- `src/opcodes_*.asm` — Opcode handlers by category (load, store, stack, call, build, misc)
- `src/pyo/*.asm` — Type implementations (int, str, list, dict, tuple, func, class, iter, bool, none, bytes, code)
- `src/marshal.asm` — .pyc marshal format deserializer
- `src/pyc.asm` — .pyc file reader (magic validation, header parsing)
- `src/builtins.asm` — Built-in functions (print, len, range, type, isinstance, etc.)
- `src/frame.asm` — Frame alloc/dealloc
- `src/object.asm` — Base PyObject ops (alloc, refcount, dealloc)
- `src/lib/` — Syscall wrappers, string/memory ops (replace libc)
- `include/` — Struct definitions (.inc): object, types, frame, opcodes, macros, marshal, builtins, errcodes

## Key Structs

Defined in `include/*.inc`. All objects start with `PyObject` (ob_refcnt +0, ob_type +8).

- **PyTypeObject** (types.inc, 192 bytes): tp_call +64, tp_getattr +72, tp_setattr +80, tp_as_number +128, tp_as_sequence +136, tp_as_mapping +144
- **PyFrame** (frame.inc): code +8, globals +16, locals +32, stack_tag_ptr +64, locals_tag_base +96, localsplus +104 (variable-size u64[])
- **PyCodeObject** (object.inc): co_consts, co_names, co_code starts at +112

## Opcode Handler Pattern

```nasm
op_example:
    ; ecx = arg (already set by eval_dispatch)
    ; rbx already advanced past 2-byte instruction word
    ; ... implementation ...
    DISPATCH          ; jmp eval_dispatch
```

Stack macros: `VPUSH_PTR reg`, `VPUSH_INT reg`, `VPUSH_FLOAT reg`, `VPUSH_NONE`, `VPUSH_BOOL reg`, `VPUSH_VAL pay, tag`, `VPOP reg` (payload only), `VPOP_VAL pay, tag`, `VPEEK reg`

## Named Frame-Layout Constants

**Never use raw numeric offsets** like `[rbp-8]`, `[rbp-16]`, `[rsp+32]` in handler code. Instead, define named `equ` constants at the top of the file and reference them as `[rbp - SA_OBJ]`, `[rsp + BO_LEFT]`, etc.

```nasm
; At top of file, after externs:
SA_OBJ    equ 8
SA_VAL    equ 16
SA_NAME   equ 24
SA_FRAME  equ 24

; In handler:
DEF_FUNC op_store_attr, SA_FRAME
    mov [rbp - SA_OBJ], rdi
    mov rsi, [rbp - SA_NAME]
```

Convention: 2-3 letter handler prefix + field name (e.g., `SA_OBJ`, `CL_NARGS`, `LA_ATTR`). Use `XX_FRAME equ N` for the `DEF_FUNC` frame size argument. For push-based layouts, use offsets relative to `rsp`.

## Python 3.12 CACHE Entries

Opcodes have trailing CACHE words that must be skipped. Key counts (each = 2 bytes):

| Opcode | CACHE entries | Skip bytes |
|--------|--------------|------------|
| LOAD_ATTR | 9 | 18 |
| STORE_ATTR | 4 | 8 |
| CALL | 3 | 6 |
| BINARY_OP | 1 | 2 |
| COMPARE_OP | 1 | 2 |

## Known Bug Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jgarzik/apython](https://github.com/jgarzik/apython) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
