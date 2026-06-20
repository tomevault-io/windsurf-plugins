---
trigger: always_on
description: Embed Lua 5.5.0 in Go applications using github.com/iceisfun/golua/v2 (the v2 line). Covers VM setup, native function binding, table manipulation, metatables, sandboxing, and provider-based capability control. A separate v1 / Lua 5.4.8 variant exists at github.com/iceisfun/golua — check the user's go.mod before applying language-level guidance.
---


# GoLua Skill

Use this when helping someone who imported GoLua and wants to embed Lua in a Go application.

> [!IMPORTANT]
> **This skill targets GoLua v2 — Lua 5.5.0** (`github.com/iceisfun/golua/v2`, branch `master`).
>
> A separate skill copy exists for the v1 line. **Before applying language-level guidance, check the user's `go.mod`:**
>
> | Module path | Version | Lua spec | Branch |
> |---|---|---|---|
> | `github.com/iceisfun/golua/v2` | v2.x | 5.5.0 | `master` (this skill) |
> | `github.com/iceisfun/golua` | v1.x / v0.5.4-golua.x | 5.4.8 | [`lua_5_4_8`](https://github.com/iceisfun/golua/tree/lua_5_4_8) (other skill copy) |
>
> Note: Go module rules give v0 and v1 the unsuffixed path; only v2+ takes a `/vN` suffix. There is no `github.com/iceisfun/golua/v1`.
>
> Both versions share the same provider architecture, sandbox model, and Go interop API — most of this skill applies either way. The Lua-language differences (5.5-only features like `global`, named varargs, `local<const>` prefix syntax, `table.create`, `error(nil)` semantics; removed math/debug APIs) are 5.5-only and will not apply to a v1 user. If you see import path `github.com/iceisfun/golua` (no `/v2`), recommend the user install the matching v1 skill copy from the `lua_5_4_8` branch.

## SKILLS

Copy-paste block for an AI assistant:

```text
SKILLS:
- GoLua is an embeddable, sandbox-first Lua runtime for Go. Two versions: v2 = Lua 5.5.0 = `github.com/iceisfun/golua/v2` (this skill); v1 = Lua 5.4.8 = `github.com/iceisfun/golua` (no `/vN` suffix — v0/v1 share one path per Go module rules). Check `go.mod` before applying language-level guidance; this skill's import paths and 5.5-only features assume v2.
- Most hosts only need five steps: parser.Parse -> compiler.Compile -> vm.New -> stdlib.Open -> v.Run.
- A fresh VM is sandboxed by default. `io`, `os`, `debug`, `time`, `chan`, `exec`, and `http` are not available unless the host enables them.
- Main host tasks:
  - run a Lua chunk once
  - compile once and reuse the compiled chunk
  - expose Go functions with vm.NewNativeFunc
  - expose Go-owned structs as Lua tables
  - call Lua functions from Go with v.ProtectedCall
  - pass tables between Go and Lua
  - raise Lua-visible errors with panic(&vm.LuaError{Value: ...})
- Native function rules:
  - Lua args are 1-indexed with v.Get(1), v.Get(2), ...
  - return values are 0-indexed with v.Set(0), v.Set(1), ...
  - return the number of Lua results
  - use v.ArgCount() for variadic functions
- Lua 5.5 features: `global` keyword for explicit global declarations, named varargs (`... args`), `local<const>`/`local<close>` prefix-attribute syntax, read-only for-loop variables, `table.create(narr, nrec)`, `error(nil)` returns `"<no error object>"`.
- Removed APIs (Lua 5.5): math.atan2, math.cosh, math.sinh, math.tanh, math.log10, math.pow, debug.setcstacklimit are all removed (nil).
- Prefer explicit type checks like IsString, IsNumber, IsTable before calling AsString, AsInt, AsTable.
- If Go owns mutable state, expose closures that capture the Go pointer. If Lua just needs data, return a plain table snapshot.
- Value constructors: vm.NewInt(int64), vm.NewFloat(float64), vm.NewString(string), vm.NewBool(bool), vm.NewTable(*Table), vm.NewNativeFunc(NativeFunc). Pre-built: vm.Nil, vm.True, vm.False.
- vm.ValueToString(val) converts any Value to a printable string.
- Tables support metatables: tbl.SetMetatable(mt) / tbl.Metatable(). Set __add, __tostring, __index, __newindex, __len, __eq, __lt, __le, __call, __concat etc. as table fields.
- Table.Get(key) is raw access (like rawget). Use v.TableGet(tbl, key) for __index-aware access (like tbl[key] in Lua). This matters for class instances.
- Source directives: `directives.Parse(src)` extracts `-- @key value` header metadata for embedder-defined annotations like `-- @tick 30s`. Pure source-level (no lexer/VM coupling), header-only, non-standard Lua (reference Lua sees them as ordinary comments). The host defines what keys mean — the package is policy-free.
```

## What You Usually Need To Know

If a user just added GoLua to their app, the useful mental model is small:

1. Parse Lua source.
2. Compile it to bytecode.
3. Create a VM.
4. Open the standard library.
5. Run the compiled chunk.

That is the core path most integrations start from.

## Lua 5.5 Language Features

GoLua implements Lua 5.5. Key language changes from 5.4:

- **`global` soft keyword**: Global variable declarations can use `global x = 10` for explicit intent with compile-time checking. Undeclared global access produces a compile warning/error when global declarations are present in the chunk.
- **Named vararg parameters**: `function f(... args)` packs variadic arguments into a table `args` with an `n` field holding the count, replacing the need for `{...}` and `select("#", ...)`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iceisfun/golua](https://github.com/iceisfun/golua) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
