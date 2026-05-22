---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Restrictions

Lua core sources live in a git submodule (`lua/` → `github.com/lua/lua` at `v5.5.0`). Do not modify files inside the submodule. Only modify files in `src/`, `include/`, and `host_tools/`.

## Project

Zephyr RTOS module integrating Lua 5.5.0 as a first-class scripting engine for embedded systems. Lua scripts run in dedicated threads with isolated heaps; IPC happens via zbus channels.

## Build Commands

Uses `just` (justfile) with West/CMake underneath. Default board: `mps2/an385`.

| Command            | Alias     | Description                                          |
| ------------------ | --------- | ---------------------------------------------------- |
| `just build`       | `just b`  | Build (`west build -d ./build -b mps2/an385 app`)    |
| `just rebuild`     | `just bb` | Rebuild without full reconfigure                     |
| `just clean`       | `just c`  | Remove build directory                               |
| `just run`         | `just r`  | Build and run in QEMU                                |
| `just config`      |           | Open menuconfig                                      |
| `just debugserver` | `just ds` | Start QEMU GDB server                                |
| `just attach`      | `just da` | Attach GDB to running debug session                  |
| `just test`        |           | Run twister test suite (`-p mps2/an385 -T samples`)  |
| `just format`      |           | Format project C/H files (excludes core Lua sources) |

### Running Tests

Tests use Zephyr's twister harness. Each sample has a `sample.yaml` defining console-based test expectations:

```sh
west twister -p mps2/an385 -T samples -O /tmp/lua_tests/
```

### Formatting

```sh
just format          # Runs clang-format on src/ include/ and samples/ C/H files
clang-format -i <file>   # Uses .clang-format (Zephyr-aligned LLVM, 8-space indent, 100-col limit)
```

## Architecture

### Module Structure

- **Lua 5.5.0 core** — Git submodule (`lua/`) from `github.com/lua/lua` at tag `v5.5.0`
- **Zephyr integration** (`src/`):
  - `luaz_utils.c` — Custom `sys_heap` allocator, `luaz_openlibs()`, minimal `require()`, `luaz_print_mem_usage()`, kernel API bindings (`zephyr.msleep`, `zephyr.printk`, `zephyr.log_*`)
  - `luaz_zbus.c` — zbus channel/observer Lua bindings (pub, read, wait_msg)
  - `luaz_msg_descr.c` — Descriptor-based Lua↔C struct conversion (helper library)
  - `luaz_repl.c` — Interactive Lua shell (enabled via `CONFIG_LUA_REPL`)
  - `luaz_fs.c` — Filesystem support: `dofile`, `loadfile`, `list`, `write_file` (enabled via `CONFIG_LUA_FS`)
  - `luaz_fs_shell.c` — Shell commands for managing Lua scripts on FS: list, cat, write, delete, run, stat (enabled via `CONFIG_LUA_FS_SHELL`)
  - `luaz_parser_stubs.c` — Parser stubs for bytecode-only builds (enabled via `CONFIG_LUA_PRECOMPILE_ONLY`)

### CMake API (`luaz.cmake`)

Single file included **twice** during a build:

1. **Pre-Zephyr** (by user CMakeLists.txt) — thread definitions + Kconfig generation
2. **Post-Zephyr** (by module CMakeLists.txt) — code generation + host luac build

**Pre-Zephyr macros** (call before `find_package(Zephyr)`):

- `luaz_define_source_thread(path)` — Define a source-embedded thread
- `luaz_define_bytecode_thread(path)` — Define a bytecode thread
- `luaz_define_fs_thread(fs_path)` — Define a filesystem-backed thread

Each generates a per-thread Kconfig fragment with `<NAME>_LUA_THREAD_STACK_SIZE`, `_HEAP_SIZE`, `_PRIORITY`.

**Post-Zephyr functions** (call after `find_package(Zephyr)`):

- `luaz_generate_threads()` — Generate all defined threads (processes `LUAZ_SOURCE_THREADS`, `LUAZ_BYTECODE_THREADS`, `LUAZ_FS_THREADS` lists)
- `luaz_add_file(path)` — Embed a `.lua` file as a C string header (via `luaz_gen.py`)
- `luaz_add_bytecode_file(path)` — Embed precompiled bytecode as a C `uint8_t[]` header (requires `CONFIG_LUA_PRECOMPILE`)
- `luaz_add_fs_file(src [name])` — Register a Lua file for embedding and writing to the FS at boot (requires `CONFIG_LUA_FS`)

### Thread Model

Each generated thread gets:

- Dedicated `sys_heap` (per-thread `CONFIG_<NAME>_LUA_THREAD_HEAP_SIZE`, defaults to `CONFIG_LUA_THREAD_HEAP_SIZE` = 32KB)
- Dedicated stack (per-thread `CONFIG_<NAME>_LUA_THREAD_STACK_SIZE`, defaults to `CONFIG_LUA_THREAD_STACK_SIZE` = 2KB)
- Custom Lua allocator backed by the thread's heap
- `luaz_openlibs(L)` called automatically — registers `require()` and preloads all enabled libraries
- A weak setup hook (`<script>_lua_setup`) for registering zbus channels/observers
- `luaz_print_mem_usage()` called after script exits — prints heap and stack usage table (requires `CONFIG_SYS_HEAP_RUNTIME_STATS` for heap, `CONFIG_INIT_STACKS` + `CONFIG_THREAD_STACK_INFO` for stack)

**Variants:**

- **Source thread** (`luaz_define_source_thread`) — script embedded as C string, parsed at startup
- **Bytecode thread** (`luaz_define_bytecode_thread`) — script precompiled, parser can be stripped
- **Filesystem thread** (`luaz_define_fs_thread`) — script loaded from FS path at runtime

### Lua API


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rodrigopex/lua_zephyr](https://github.com/rodrigopex/lua_zephyr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
