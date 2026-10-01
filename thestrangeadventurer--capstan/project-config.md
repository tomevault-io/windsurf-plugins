---
trigger: always_on
description: Capstan is a CLI LLM agent, similar to OpenCode and Claude Code.
---

# AGENTS.md

Capstan is a CLI LLM agent, similar to OpenCode and Claude Code.

## Build

### Quick start

```sh
./build.sh
```

Cross-platform: macOS (arm64 / x86_64) and Linux. The script checks system
dependencies, builds vendored ncurses + Lua, then compiles the project.

For a project-only rebuild (ncurses + Lua already built):

```sh
make clean && make
```

`compile_commands.json` is stale — ignore it, use the Makefile.

### Build principles

**Static linking** — ncurses and Lua are linked as `.a` archives.
- No runtime dependencies besides `libcurl` (used for LLM API calls).
- The binary is a single Mach-O / ELF file — portable, no shared libs needed.
- ncurses: `--enable-widec` (wide-char for Unicode), `--with-termlib`
  (separates terminal constants like `_COLS` into `libtinfow.a`).
  `--without-shared` — only static `.a` archives are needed; `.so`/`.dylib` are skipped.

**Single compilation unit** — all `src/*.c` compiled in one `gcc` invocation.
- No per-file `.o` objects, no incremental build.
- Simpler Makefile, faster link step.
- Trade-off: full recompile on any change (acceptable for a project this size).

**Vendored, not system packages** — ncurses and Lua built from source in `vendor/`.
- Avoids version mismatches (e.g. Lua 5.4 vs 5.5, ncurses API changes).
- `libtinfow.a` must come from the same ncurses build — system ncurses may
  not provide it (`--with-termlib` is non-default).
- Build is reproducible — `build.sh` can be re-run after OS switch or arch change.

**Cross-platform** — `build.sh` auto-detects OS at runtime.
- macOS: `sysctl -n hw.ncpu`, Xcode CLT, libcurl via SDK `.tbd` stubs.
- Linux: `nproc`, `build-essential`/`libcurl-dev` plus `infocmp`
  (`ncurses-bin` on Debian/Ubuntu) via `apt`/`dnf`.
- ncurses and Lua auto-detect the platform in their own `configure`/`Makefile`.

### Makefile breakdown

```
CC      = gcc                     # Apple Clang on macOS, GCC on Linux
CFLAGS  = -std=gnu99 -Wall -Wextra -Werror -D_POSIX_C_SOURCE=200112L
          + -Iinclude -Ivendor/... (headers)

LDFLAGS = vendor/lua-5.5.0/src/liblua.a
        + vendor/ncurses-install/lib/libncursesw.a
        + vendor/ncurses-install/lib/libtinfow.a
        + -lm -lcurl
          \______________________/  \__/  \___/
                               |        |      |
                     vendored static   math  system
```

- Static `.a` files are passed as **direct paths** (not `-L`/`-l`) to avoid
  accidentally picking up wrong system libraries.
- `-D_POSIX_C_SOURCE=200112L` — enables POSIX.1-2001 (`fork`, `pipe`,
  `sigaction`, etc.).
- `ncursesw` — the wide-char variant (`wchar_t`, Unicode).
- `tinfow` — terminfo library separated by `--with-termlib`; contains
  `_COLS`, `cur_term`, and other terminal capability constants.
- `-lcurl` — only dynamic dependency (linked from system).
- `-lm` — math library, pulled in by Lua.

### Directory layout after build

```
vendor/ncurses-install/    # ncurses headers + static .a libs (gitignored)
vendor/lua-5.5.0/src/*.o  # Lua object files (gitignored)
build/capstan              # final binary (gitignored)
```

## Architecture — non-obvious

### Architecture policy

When adding or changing behavior that affects multiple code paths:

- Prefer one canonical implementation of the policy or business rule over
  duplicated partial implementations.
- If the behavior must exist in multiple layers, define which layer owns the
  policy and which layers are adapters, caches, fallbacks, or compatibility
  shims.
- Prefer integrating with the existing configuration model instead of adding
  hidden constants or parallel configuration paths.
- Before implementing, identify all affected data paths: UI, model context,
  logs, persisted state, command-line mode, tests, and plugin/runtime APIs.
- Define the failure mode explicitly: fail open, fail closed, preserve old
  behavior, or surface an error.
- Add tests for the intended behavior and for important false positives or
  regressions.

Avoid fixing only the observed symptom when the same rule clearly applies
across multiple paths. First find the owner of the rule, then route callers
through that owner.

### Init order (critical)

`plugins_init()` in `src/plugins.c`:
1. `luaL_newstate` → `luaL_openlibs`
2. `package.path` is configured with `~/.config/capstan/?.lua`
3. Embedded Lua modules are registered in `package.preload`
4. `http_init(L)` — registers global `http = {get, post, post_stream, ...}`
5. `agent_init(L)` — registers global `agent = {append, set_info, ...}`
6. `tools_init(L)` — registers built-in tool helpers
7. `mcp_init(L)` — registers global `mcp = {spawn, send, recv, alive, kill}`
8. `capstan` runtime paths are registered
9. User config and persisted runtime state are loaded into `capstan`
10. `permit`, `log`, and `popup` globals are registered
11. The embedded system prompt plus project instructions/skills are loaded
12. `agent/runtime.lua` is loaded — side-effect: sets `_G.agent_entry`

Plugins are loaded AFTER this — they can use the registered globals.

### Message flow (the tricky part)

After a non-empty Enter (command or plain text, or empty Enter with buffered results pending):
```
add_message(ui_text, raw_text, MSG_USER)  // user/plugin text

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theStrangeAdventurer/capstan](https://github.com/theStrangeAdventurer/capstan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
