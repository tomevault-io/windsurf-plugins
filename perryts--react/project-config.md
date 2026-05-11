---
trigger: always_on
description: This file documents every non-obvious decision, bug, and fix made during the implementation of `perry-react`. It exists so future sessions can pick up where this left off without re-discovering the same things.
---

# CLAUDE.md — perry-react engineering notes

This file documents every non-obvious decision, bug, and fix made during the implementation of `perry-react`. It exists so future sessions can pick up where this left off without re-discovering the same things.

---

## Project layout

```
perry-react/
  src/index.ts          — the entire renderer: createElement, hooks, reconciler,
                          element→widget mapping, createRoot
  demo/
    src/main.tsx        — entry point: createRoot + root.render(<App />)
    src/App.tsx         — WorkBench demo component (14 test sections)
    package.json        — packageAliases wiring
    main                — compiled binary (gitignored)
    kanban-board.jsx    — web-first kanban demo (NOT compilable with Perry)
    kanban/             — native-adapted kanban board demo
      src/main.tsx      — entry point (1400×800 window)
      src/App.tsx       — kanban board: 5 columns, cards, move/add/delete/view
      package.json      — packageAliases wiring
  package.json          — perry-react package manifest
```

The renderer (`src/index.ts`) is a single file by design. Perry's native module system requires a single entry point and the whole thing compiles to one `.o`.

---

## How the import alias works

`demo/package.json`:
```json
{
  "perry": {
    "packageAliases": {
      "react":             "perry-react",
      "react/jsx-runtime": "perry-react",
      "react-dom":         "perry-react",
      "react-dom/client":  "perry-react"
    }
  }
}
```

Perry reads this in `compile.rs` around line 934–937. Before codegen, the compiler substitutes the alias target wherever it sees the aliased package name. This is the same mechanism used for `@prisma/client → perry-prisma` etc. No source code changes needed in user components.

`perry-react` is declared as a native module (`"nativeModule": true` in its `package.json`). Perry adds it to the same `NATIVE_MODULES` list as `perry/ui`, which means it's compiled and linked as native Perry TypeScript, not run through V8.

---

## JSX transform path

Perry's parser handles `.tsx` natively (SWC-based). The JSX lowering is in `perry/crates/perry-hir/src/lower.rs`, function `lower_jsx_element` (~line 10209).

```
<div style={s}>              →  Expr::Call {
  <h1>text</h1>                   callee: ExternFuncRef("jsxs"),
  <p>{count}</p>                  args: [
</div>                              "div",
                                    Expr::Object([
                                      ("style", s),
                                      ("children", Expr::Array([
                                        Expr::Call(ExternFuncRef("jsx"), ["h1", ...]),
                                        Expr::Call(ExternFuncRef("jsx"), ["p",  ...]),
                                      ]))
                                    ])
                                  ]
                                }
```

- Single child → stored directly as `children` value (not wrapped in array)
- Multiple children → `Expr::Array([...])` for the `children` prop
- `jsx` for 0–1 children, `jsxs` for 2+ children (React convention)

`ExternFuncRef("jsx")` resolves to `__wrapper_jsx` at link time — the wrapper Perry generates for every exported function in a native module. The wrapper ABI is `(i64 closure_ptr, f64 arg0, f64 arg1, ...) -> f64`.

---

## NaN-boxing in Perry

Every JS value is stored as a 64-bit NaN-boxed float (`f64`). The encoding:

```
Normal float:   standard IEEE 754 f64 (exponent not all-1s)
Pointer:        0x7FFD_0000_0000_0000 | (ptr & 0x0000_FFFF_FFFF_FFFF)
String:         0x7FFC_0000_0000_0000 | str_ptr
Integer:        0x7FFE_0000_0000_0000 | (i32 as u32)
Undefined:      0x7FFF_8000_0000_0001
Null:           0x7FFF_8000_0000_0002
Bool true:      0x7FFF_0000_0000_0001
Bool false:     0x7FFF_0000_0000_0000
```

Key helpers in `codegen.rs`:
- `inline_nanbox_pointer(builder, I64_ptr) -> F64` — masks lower 48 bits, ORs POINTER_TAG, bitcasts
- `ensure_i64(builder, F64_val) -> I64` — strips top 16 bits via `& 0x0000_FFFF_FFFF_FFFF`
- `ensure_f64(builder, val) -> F64` — raw bitcast I64→F64 or identity; **does NOT NaN-box**

`ensure_i64` is safe for both properly NaN-boxed pointers and raw-bitcast subnormals because the mask strips the tag either way. This is why the array fix (Fix 5 below) doesn't break existing callers.

---

## The Perry compiler fixes

Fixes 1–5 are in `perry/crates/perry-codegen/src/codegen.rs`.
Fix 6 is in `perry/crates/perry-codegen/src/expr.rs`.

### Fix 1 — `js_native_call_method` receiver (~line 28179)

**Problem**: When compiling `obj.someMethod(args)`, the object value was passed raw to `js_native_call_method`. If the object was an I64 (raw pointer), the runtime's `value.is_pointer()` check failed because the POINTER_TAG wasn't set.

**Fix**:
```rust
let obj_val = if obj_val_type == types::I64 {
    inline_nanbox_pointer(builder, obj_val_raw)
} else {
    obj_val_raw
};
```

### Fix 2 — `js_native_call_method` arguments (~line 28220)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PerryTS/react](https://github.com/PerryTS/react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
