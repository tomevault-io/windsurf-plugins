---
trigger: always_on
description: `mirror_bridge` is a C++26 reflection-based binding generator: it inspects C++ structs/classes at
---

# AGENTS.md

`mirror_bridge` is a C++26 reflection-based binding generator: it inspects C++ structs/classes at
compile time (via P2996 reflection) and emits zero-overhead Python, Lua, and JavaScript bindings —
no hand-written binding code. **The one critical constraint: it requires a C++26 reflection
compiler** — either stock **GCC 16+** (`g++ -std=c++26 -freflection`) or Bloomberg's
[clang-p2996](https://github.com/bloomberg/clang-p2996) (with libc++ and the `<meta>` header).
A pre-reflection distro compiler will fail with "no member named 'meta'" or an unknown
`-freflection` flag. Unless your host already has GCC 16+ or clang-p2996, compile inside the
project's Docker container (or a GitHub Codespace). The CLI auto-detects whichever compiler is
present; override with `MB_CXX`. Editing, searching, and reading code on the host is always fine.

Note: P3394 field annotations (`[[=exclude{}]]`, `[[=readonly{}]]`) are currently clang-p2996 only.
On GCC they are silently ignored (all members bound) until GCC implements P3394.

## Environment setup

The reflection compiler lives in the Docker image. Prefer pulling the pre-built image over building
from source (~45–90 min).

```bash
# Pull + tag the pre-built image (interactive helper, choose option 1)
./start_dev_container.sh

# Or pull directly
docker pull ghcr.io/franciscothiesen/mirror_bridge:latest
docker tag  ghcr.io/franciscothiesen/mirror_bridge:latest mirror_bridge:latest
```

`start_dev_container.sh` creates a persistent container named `mirror_bridge_dev` with the repo
mounted at `/workspace` (host edits are immediately visible inside, and vice versa).

To run a command inside the container **non-interactively** (scriptable for an agent):

```bash
# One-off, throwaway container with the repo mounted:
docker run --rm -v "$(pwd):/workspace" -w /workspace mirror_bridge:latest \
    bash -lc './tests/run_all_tests.sh'

# Or exec into the long-lived dev container if it's already running:
docker exec -w /workspace mirror_bridge_dev bash -lc 'tools/mirror_bridge version'
```

If a binary can't find libc++ at runtime, set `LD_LIBRARY_PATH=/usr/local/lib` (see failure modes).

## Common tasks

The CLI is the bash script `tools/mirror_bridge` (run it as `tools/mirror_bridge` or, from inside an
example dir, `../../tools/mirror_bridge`). All commands below assume you are inside the container.

**Generate bindings from headers (auto-discovery):**

```bash
# Single language
tools/mirror_bridge generate src/ --module my_module --lang python

# Choices for --lang: python | lua | js | all
tools/mirror_bridge generate src/ --module my_module --lang all

# Useful flags: -o/--output DIR (default build/), -k/--keep-generated,
# -v/--verbose, -f/--force, -I DIR (extra include dir), --pch [PATH]
tools/mirror_bridge generate src/ --module my_module --lang python -I include/ -v
```

`generate` scans every `.hpp`/`.h` under `src_dir`, binds all discovered struct/class definitions,
emits a `<module>_<lang>_binding.cpp`, compiles it, and (unless `-k`) deletes the generated `.cpp`.

**Build a precompiled header (3–6x faster builds):**

```bash
tools/mirror_bridge pch --output build/ --type release   # or --type debug
tools/mirror_bridge generate src/ --module my_module --lang python --pch   # auto-detects build/*.gch
```

**Build a hand-written binding file:**

```bash
tools/mirror_bridge build my_bindings.cpp --lang python -I src/ -o build/
```

**Run the full test suite:**

```bash
./tests/run_all_tests.sh
```

It builds every `*.cpp` binding under `tests/` and runs all `test_*.py`, `test_*.js`, `test_*.lua`,
and `test_*.sh`. V8 tests are non-blocking. The script sets `LD_LIBRARY_PATH`, `PYTHONPATH`, and
`LUA_CPATH` for you.

**Run a single test** (no per-test target exists; do it manually inside the container):

```bash
# 1. Build the binding the test imports (Python example):
clang++ -std=c++2c -freflection -freflection-latest -stdlib=libc++ \
    -I. -Itests/<dir> -fPIC -shared $(python3-config --includes --ldflags) \
    tests/<dir>/<module>_binding.cpp -o build/<module>.so
# 2. Run it:
LD_LIBRARY_PATH=/usr/local/lib PYTHONPATH=build python3 tests/<dir>/test_<name>.py
```

**Regenerate after header changes / inspect the binding surface:**

```bash
tools/mirror_bridge generate src/ --module my_module --lang python -f   # force rebuild
tools/mirror_bridge diff src/                 # show added/removed members vs. last snapshot
tools/mirror_bridge diff src/ --update        # update snapshot non-interactively (CI-friendly)
tools/mirror_bridge watch src/ --module my_module --lang python   # poll + auto-recompile
```

**One-off compile of a binding (the canonical command the CLI runs under the hood):**

```bash
# Python:
clang++ -std=c++2c -freflection -freflection-latest -stdlib=libc++ \
    -I. -I<src_dir> -I<project_root> -fPIC -shared -O3 -DNDEBUG \
    -Wno-deprecated-declarations $(python3-config --includes) \
    binding.cpp -o build/my_module.so $(python3-config --ldflags)

# Lua:  add  -I/usr/include/lua5.4 -llua5.4   and output .so
# JS:   add  -I/usr/include/node               and output .node
```

The four flags that must always be present: `-std=c++2c -freflection -freflection-latest

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FranciscoThiesen/mirror_bridge](https://github.com/FranciscoThiesen/mirror_bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
