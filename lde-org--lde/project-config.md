---
trigger: always_on
description: `lde` is a package manager and toolkit for Lua, written in Lua and running on LuaJIT. It manages project-local dependencies, runs Lua programs, and compiles them into single executables.
---

# lde

`lde` is a package manager and toolkit for Lua, written in Lua and running on LuaJIT. It manages project-local dependencies, runs Lua programs, and compiles them into single executables.

## Repo Structure

```
packages/
  lde/          # The CLI binary itself (entry: src/init.lua)
  lde-core/     # Core library: Package, Lockfile, runtime, install logic
  lde-test/     # Built-in test framework
  ansi/clap/env/fs/git/http/json/path/process/semver/util/  # Internal packages
  sea/          # Single-executable assembly (compiles bundles into binaries)
  archive/      # Archive extraction support
  luarocks/     # LuaRocks integration
  rocked/       # Rockspec support
schemas/        # JSON schema for lde.json
tests/          # Top-level integration test fixtures (e.g. some-package/)
```

## How `require()` Paths Are Resolved

When `lde run` (or `lde test`) executes a script, it sets `package.path` and `package.cpath` to point at the package's `target/` directory:

```
target/?.lua
target/?/init.lua
target/?.so  (or .dll / .dylib)
```

`target/` is populated by `lde install` / `lde run` (which calls `installDependencies` first). Each dependency is installed as a symlink (or copy if it has a build script) at `target/<alias>`. So `require("json")` resolves to `target/json/init.lua`, which is a symlink to `packages/json/src/init.lua`.

The key: **the require name is the key in `lde.json` `dependencies`, not the package's `name` field.** You can alias a package by using a different key.

## The `tests` Special `require()` Path

During `lde test`, the runner automatically exposes the package's `tests/` directory as `target/tests` (symlinked or copied). This means test files can `require("tests.lib.something")` to share helpers across test files.

Example from `packages/lde/tests/main.test.lua`:

```lua
local ldecli = require("tests.lib.ldecli")
```

This resolves to `tests/lib/ldecli.lua` via the `target/tests` symlink.

## Test Framework (`lde-test`)

Test files must match `**/*.test.lua`. See `packages/lde-test` for the full API.

```lua
local test = require("lde-test")

test.it("does something", function()
  test.equal(a, b)       -- assertions take no message argument
  test.truthy(x)
  test.deepEqual(t1, t2)
end)
```

Run tests: `lde test` (from a package dir, or from repo root to run all packages).

## `lde.json` Config

```jsonc
{
  "name": "my-package",
  "version": "0.1.0",
  "bin": "src/main.lua",          // optional, overrides default entry (target/<name>/init.lua)
  "engine": "lde",                // "lde" (default), "lua", or "luajit"
  "scripts": { "build": "..." },  // runnable via `lde <name>` or `lde run <name>`
  "dependencies": {
    "json":    { "path": "../json" },           // local path
    "hood":    { "git": "https://...", "commit": "abc123" },
    "semver":  { "version": "1.0.0" },          // registry
    "mylib":   { "luarocks": "luafilesystem" }, // luarocks
    "archive": { "archive": "https://.../x.zip" },
    "winapi":  { "git": "...", "optional": true }
  },
  "devDependencies": { ... },
  "features": {
    "windows": ["winapi"]  // optional deps enabled per platform/feature flag
  }
}
```

Lockfile is `lde.lock`. The `target/` directory is the build output — never commit it.

## Build System

- `lde run` / `lde test` both call `pkg:build()` + `pkg:installDependencies()` automatically.
- **Build script**: if `build.lua` exists at the package root, it's executed with `LDE_OUTPUT_DIR` set to the output path. Otherwise, `src/` is symlinked directly into `target/<name>`.
- `target/.installed` stores an FNV1a hash of `lde.lock` as a fast-path cache — if it matches, install is skipped entirely.

## Updating the `lde` Binary

After making changes to any package source, rebuild the binary:

```sh
cd packages/lde
lde compile
```

This outputs `packages/lde/lde` (or `lde.exe` on Windows). To install it, copy it to `~/.lde/lde`.

**Important:** Tests in `packages/lde/tests/` run the actual `lde` CLI binary via `env.execPath()`. If those tests are failing after source changes, you need to recompile and replace the binary first.

## Runtime Isolation (`lde-core.runtime`)

`lde run` / `lde test` execute scripts in an isolated environment:

- `package.loaded` is cleared of non-builtins before and restored after each run.
- A fresh `_G` metatable is created per execution (`setfenv`).
- `ffi.cdef` is patched to silently ignore "attempt to redefine" errors (safe for repeated runs in the same process).
- `package.path`/`cpath` are restored after execution.

This means multiple `lde run` calls in the same process don't pollute each other's module state.

## Key Packages

| Package    | Purpose                                                                  |
| ---------- | ------------------------------------------------------------------------ |
| `lde-core` | `Package`, `Lockfile`, install/build/run/test/compile logic              |
| `lde-test` | Test framework (`require("lde-test")` in test files)                     |
| `clap`     | CLI argument parsing (`args:option()`, `args:flag()`, `args:pop()`)      |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lde-org/lde](https://github.com/lde-org/lde) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
