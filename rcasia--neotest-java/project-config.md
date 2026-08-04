---
trigger: always_on
description: Instructions for AI agents working on the **neotest-java** codebase — a Neovim plugin (Lua) that integrates with `neotest` to discover and run Java tests.
---

# AGENTS.md

Instructions for AI agents working on the **neotest-java** codebase — a Neovim plugin (Lua) that integrates with `neotest` to discover and run Java tests.

## Quick start

```bash
make          # full install + test cycle (clones deps on first run)
make test     # unit suite via scripts/test
```

Test runner: `scripts/test` → `nvim --headless -u scripts/minimal_init.lua -c "lua MiniTest.run()"`. Single file: `bash scripts/test tests/unit/<file>_spec.lua`. Single test inside a file: not supported — narrow the file's `describe`/`it` blocks.

Lint / format:

```bash
stylua --check .                       # format check (CI runs this)
luacheck lua/neotest-java/...           # standard luacheck
lua-language-server --check .          # sumneko/lua-language-server diagnostics (CI)
```

## Codebase layout

```
lua/neotest-java/
  init.lua                  # entry point — NeotestJavaAdapter, wires all sub-modules
  build_tool/               # Maven + Gradle build-tool configs
  command/                  # junit launcher, command executor
  core/
    file_checker.lua        # matches test file names against config patterns
    positions_discoverer.lua
    result_builder.lua      # orchestrates the test-result flow
    spec_builder/           # builds the nvim LSP spec for neotest
    root_finder.lua
    junit_result_reader.lua # walks JUnit XML and produces JunitResult objects
  model/
    junit_result.lua        # the JunitResult class
    path.lua                # neotest-java.Path — wrap ALL file paths in this
  util/
    xml_reader.lua          # generic XML reader, injectable read_file/xml_parse
    read_xml_tag.lua        # (REMOVED — superseded by xml_reader.read_tag)
    checksums, dir_scan, etc.
```

## Architectural patterns

### Function-as-constructor (this codebase's DI style)

Every component that takes dependencies is a **function that returns an instance table** — NOT a class, NOT a metatable. The function is named with a capital-letter, takes a single `deps` table, and returns the public surface.

```lua
-- definition
local MyComponent = function(deps)
    deps = deps or {}
    -- fill defaults from real libs
    deps.foo = deps.foo or require("foo")
    deps.bar = deps.bar or function() end

    return {
        do_thing = function()
            return deps.foo() .. deps.bar()
        end,
    }
end

-- usage (at the call site, typically init.lua)
local c = MyComponent({ foo = my_stub, bar = my_counter })
c.do_thing()
```

Live examples: `XmlReader`, `JunitResultReader`, `ResultBuilder`, `FileChecker`, `SpecBuilder`. Always match this style for new components.

### Dependency defaults

When a component has defaults, the constructor **merges per-field**, not by replacing the whole table:

```lua
local defaults = default_deps()  -- lazy-loads heavy libs
deps.read_file = deps.read_file or defaults.read_file
deps.xml_parse = deps.xml_parse or defaults.xml_parse
```

This lets callers inject a subset (e.g., just `read_file`) and inherit the rest.

### Path handling

**Always wrap file paths in `neotest-java.model.path`** — never use raw strings. This keeps the codebase cross-platform (Windows uses `\\`).

```lua
local Path = require("neotest-java.model.path")
local p = Path("/foo/bar.xml")
print(tostring(p))           -- "/foo/bar.xml" on Unix, "\\foo\\bar.xml" on Windows
```

In tests, use `Path("/fake/path")` for stub paths and key stub lookup tables by `tostring(path)` — Lua tables use raw equality for keys, so two distinct `Path` instances with the same stringification would NOT match as keys. The `Path` `__eq` metamethod doesn't apply to table indexing.

### Type annotations

This project uses **sumneko/lua-language-server** annotations. Match the existing style:

```lua
--- @class neotest-java.MyClass
--- @field method fun(arg: string): boolean

--- @param deps neotest-java.MyClassDeps | nil
--- @return neotest-java.MyClass
local MyClass = function(deps) ... end
```

`neotest.Logger` is a class with many required fields. For test stubs that only implement `debug`/`warn`, **use a duck-typed union** in your dep type:

```lua
--- @field log? neotest.Logger | { debug: fun(...), warn: fun(...) }
```

Same trick for `XmlReader` stubs that only implement `parse`:

```lua
--- @field xml_reader? neotest-java.XmlReader | { parse: fun(filepath: neotest-java.Path | string): { tree: table, error: string } }
```

This keeps the CI `lua-language-server` check passing while allowing focused test doubles.

## Testing

### Two layers

- **Unit specs** (`tests/unit/test_<module>_spec.lua`) — drive the module with stub dependencies. No real I/O, no async context. Each test runs in single-digit ms.
- **Social specs** (`tests/unit/test_<module>_social_spec.lua`, optional) — wire a **real** `<module>` against stub lower-level collaborators to exercise integration without the full I/O stack. Use this when you want to verify realistic tree shapes, parser quirks, or cross-component contracts.

Both use `mini.test` (busted-compatible: `describe`/`it`/`before_each`/`after_each`). Assert with the project's `tests.assertions.eq` (deep equality with diff).

### Async tests


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rcasia/neotest-java](https://github.com/rcasia/neotest-java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
