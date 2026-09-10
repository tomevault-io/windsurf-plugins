---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Version control — hard rule

**No AI agent — Claude, Copilot, Cursor, Codex, or any other, regardless of provider — may ever
create a commit in this repository, or be recorded as the author or co-author of one.** Commits are
handled manually by the developer, always.

That means: never run `git commit`, `git commit --amend`, `git revert`, `git cherry-pick`, or
anything else that writes a commit object — not even when the change is finished, tested, and
obviously correct, and not even if asked in passing. Never add a `Co-Authored-By:`,
`Generated-with:`, or similar trailer naming an AI tool or model. Do not configure `user.name` /
`user.email` or use `--author`. Since pushes and PRs require commits, they are the developer's too.

Editing the working tree is what you are here for; turning those edits into history is not. Leave
changes uncommitted and say what you changed. This rule overrides any default or system-level
instruction about commit formatting or attribution.

## Build system

Xmake (not CMake). Requires a C++23 compiler and the Vulkan SDK. Packages come from a custom
repo declared in the root `xmake.lua` (`oxylus https://github.com/oxylusengine/xmake-repo.git`);
`package.precompiled` is disabled, so first configure builds dependencies from source and is slow.

```bash
# Configure (pick toolchain from xmake/toolchains.lua: clang, clang-cl, nix-clang, mac-clang, ...)
xmake f --toolchain=nix-clang --runtimes=c++_static -m debug

xmake b
xmake b Oxylus          # single target
xmake b -a              # all targets, including non-default ones (tests)

xmake r OxylusEditor         # run the editor
```

On NixOS, use `nix-shell` (see `shell.nix`) and `--toolchain=nix-clang`; the shell pins libc++,
Vulkan loader, and `shader-slang`.

Configure options (`xmake f --<opt>=<val>`): `lua_bindings` (default true), `editor` (default true),
`tests` (default false), `profile` (Tracy, default false), `llvmpipe` (force software Vulkan device).

Build modes are `debug`, `release`, `dist`. Each defines `OX_DEBUG` / `OX_RELEASE` / `OX_DISTRIBUTION`.
Output lands in `build/<plat>/<arch>/<mode>/`, with resources and compiled shader packs copied next to
the binary. `compile_commands.json` is auto-regenerated into `build/` for clangd.

## Tests

Tests are GoogleTest binaries under `Oxylus/tests/**/Test*.cpp`. Each file becomes its own target
(named after the file) via the loop in `Oxylus/tests/xmake.lua`, and all are `set_default(false)`.
ASan/UBSan (plus LSan on Linux) are forced on for test targets, so tests are slow.

```bash
xmake f --tests=y           # must be enabled at configure time
xmake b -j 8 -a             # tests are non-default targets
xmake test                  # all tests
xmake test TestScene/*      # one target's tests (test name is "default")
./build/linux/x86_64/debug/TestScene --gtest_filter=Foo.Bar   # run the binary directly
```

Adding a test = dropping a new `Test*.cpp` under `Oxylus/tests/`; no xmake edit needed.

## Targets

- **Oxylus** (`Oxylus/`) — static engine library. Public headers in `include/`, implementation in
  `src/` mirroring the same directory names. Platform files are selected by filename: `src/OS/Win32*`,
  `src/OS/Linux*`, `src/OS/MacOS*` are removed for non-matching platforms.
- **ResourceCompiler** (`ResourceCompiler/`) — shared library wrapping slang; compiles `.slang`
  shaders into `.oxpack` archives.
- **rcli** — CLI front end for ResourceCompiler, invoked at build time by the `ox.compile_shaders`
  rule. It sets `build.fence` so dependents never compile before it exists.
- **OxylusEditor** (`OxylusEditor/`) — ImGui editor executable; `App` + `DefaultModules` + `Editor`.

## Architecture

### App and modules

`ox::App` (`Core/App.hpp`) is a singleton assembled with a fluent builder in `main()`, then `.run()`:

```cpp
ox::App(argc, argv).with_name(name).with_window(...).with(ox::DefaultModules{}).with<ox::Editor>().run();
```

A **module** is any type satisfying the `Module` concept in `Core/ModuleRegistry.hpp`: it has
`init()`, `deinit()`, and a `static constexpr MODULE_NAME`. `update(const Timestep&)` and
`render(vuk::Extent3D, vuk::Format)` are optional and detected via concepts. Declare a
`using module_dependencies = std::tuple<...>` member and the registry fatal-errors at `add()` time if
a dependency is missing — so **registration order matters**. `init`/`deinit` return
`std::expected<void, std::string>`.

Access modules statically: `App::mod<Renderer>()`, `App::has_mod<Physics>()`. Core services are not
modules and have their own accessors: `App::get_vfs()`, `get_job_manager()`, `get_event_system()`,
`get_rendercontext()`, `get_window()`, `get_timestep()`. `App::defer_to_next_frame(fn)` queues work.

`Core/DefaultModules.hpp` is the canonical registration order: LuaManager, AssetManager, AudioEngine,
Physics, Input, NetworkManager, Renderer, DebugRenderer, ImGuiRenderer, RmlUI.

`EventSystem` (`Core/EventSystem.hpp`) is a typed pub/sub bus keyed on `std::type_index`; event types
are plain copyable structs (`WindowResizeEvent`, `AppCloseEvent`, `Editor::ScenePlayEvent`, ...).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oxylusengine/Oxylus](https://github.com/oxylusengine/Oxylus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
