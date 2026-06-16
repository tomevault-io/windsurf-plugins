---
trigger: always_on
description: Rules for working in **bun-win32** — a monorepo of zero-dependency Win32 FFI bindings for [Bun](https://bun.sh) on Windows. Follow them exactly.
---

# AGENTS

Rules for working in **bun-win32** — a monorepo of zero-dependency Win32 FFI bindings for [Bun](https://bun.sh) on Windows. Follow them exactly.

Every system DLL is its own `@bun-win32/{name}` package (`packages/{name}/`) that binds the DLL's exports through `bun:ffi`. There is no marshaling layer: after the first call resolves a symbol via `dlopen`, every subsequent call is a direct native pointer invocation.

> **The deep playbook is [`PROMPT.md`](./PROMPT.md).** It is the authoritative, step-by-step guide for generating and completing a package (FFI type mapping, nullability, examples, audits). This file is the day-to-day operating manual: the rules, the layout, the toolchain, and the gates. When the two overlap, they agree; when you need detail, read `PROMPT.md`. Each package also ships an `AI.md` documenting its own binding contract.

---

## Core Principles

- **Plan before implementing.** Read and understand the problem, the existing code, and the surrounding context before writing anything. Do not guess at what code does — read it.
- **No fabrication — verify every claim.** This repo binds a real OS. Never guess a signature, type, nullability, or export. Verify against `dumpbin` (the source of truth for what exists), the Microsoft Learn docs page, and the Windows SDK header. Incorrect bindings segfault; incorrect information is worse than none. If you do not know, say so.
- **Minimal, surgical diffs.** Change only what the task requires. Do not "clean up," reformat, or refactor code you were not asked to touch. Do not mutate already-shipped bindings on a hunch.
- **No premature abstraction.** No helpers, wrappers, or utilities unless explicitly requested. Three similar lines beat a clever abstraction. Public method bodies are deliberately one line each.
- **Verify at every step.** After every meaningful change, prove it works: run the file (`bun run …`), type-check (`bunx tsc --noEmit`), and run the relevant audit. Do not pile changes on a broken state. Do not move on until the current step is verified.

---

## Repository Layout

```
packages/
  core/          @bun-win32/core      — the only non-binding package: Win32 base class,
                                         shared types (DWORD, HANDLE, …), runtime/extensions.ts
  template/      @bun-win32/template  — scaffold; WIN32_CLASS placeholders, no example/ dir
  all/           @bun-win32/all       — aggregator: depends on every package, re-exports each
                                         PascalCase class; home of the flagship example/ demos
  bun-win32/     bun-win32            — unscoped alias; `export * from '@bun-win32/all'`
  terminal/      @bun-win32/terminal  — high-performance terminal rendering engine (binds kernel32)
  {name}/        @bun-win32/{name}    — one package per system DLL (advapi32, kernel32, user32, …)
scripts/         repo automation (see Commands) — run with `bun run scripts/{name}.ts`
PROMPT.md        the package-generation playbook
biome.json       formatter config (Biome is the formatter; linter & assist are off)
bunfig.toml      pins linker = "hoisted"
tsconfig.json    strict; shared by every package
```

There are 117 packages. Class names are PascalCase; a few preserve native DLL casing — `OpenGL32`, `GLU32`, `Ws2_32`, `Xaudio2_9`, `Xinput1_4`, `Xinput9_1_0` — and `opengl32`/`glu32` keep native function names (`glBegin`, `gluSphere`).

### Per-package file layout

```
packages/{name}/
  index.ts                 default-import the class, re-export types — e.g. for psapi:
                             import Psapi from './structs/Psapi'; export * from './types/Psapi'; export default Psapi;
  structs/{Class}.ts       Symbols (FFI decls) + public static methods
  types/{Class}.ts         type aliases, enums, constants (re-export shared types from core)
  example/                 runnable demos (≥ 2: one creative, one professional)
  AI.md  README.md  package.json  tsconfig.json
```

No other files or directories in a package. `core` exports `{ Win32 }` (named) instead of a default.

---

## Architecture: the `Win32` Base Class

Every package subclass extends `Win32` from `@bun-win32/core`. You do **not** call `dlopen` yourself.

1. `protected static override readonly name = '{name}.dll';`
2. Override `Symbols` with the FFI declarations: `as const satisfies Record<string, FFIFunction>`.
3. Expose `public static` methods whose body is always one line: `return {Class}.Load('ExportName')(args);`

- **`Load(method)`** — lazy. On first call, `dlopen`s **only that one export**, then memoizes the native function with `Object.defineProperty` (non-configurable). Zero startup cost; each export binds at most once.
- **`Preload(methods?)`** — eager. Binds all (or a named subset of) symbols up front for hot paths; skips already-bound ones. Destructure **after** `Preload`, or you capture the lazy wrapper instead of the native function.
- **`core/runtime/extensions.ts`** is imported for its side effect: it adds a non-enumerable `.ptr` getter to `ArrayBuffer`, `Buffer`, `DataView`, and the `TypedArray`s. That is why examples write `buffer.ptr`.

---

## Toolchain


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ObscuritySRL/bun-win32](https://github.com/ObscuritySRL/bun-win32) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
