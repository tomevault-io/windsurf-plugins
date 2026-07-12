---
trigger: always_on
description: DuckDB extension that adds OAuth 2.1 / OIDC authentication and a
---

# quack-oauth — development guide

DuckDB extension that adds OAuth 2.1 / OIDC authentication and a
claims-driven authorization model to the `duckdb-quack` client/server
protocol. See `requirements.md` and `architecture.md` (arc42) for the
*what* and *why*. See `docs/IMPLEMENTATION.md` for the *how we build
and test it* (TDD discipline, test layering, real-world IdP plan,
static-linkage policy, slice-by-slice plan). This file is the local
dev / workflow guide — start here when you need to know *how* to
build, test, or extend.

Target name: **`quack_oauth`** (snake_case, used as `INSTALL quack_oauth`
/ `LOAD quack_oauth` and as `TARGET_NAME` in `CMakeLists.txt`).

## Knowledge updates

This file is a living document. When a session of work uncovers
something that future sessions would want to know — a persistent
error and its fix, a non-obvious DuckDB C++ API behavior, a vcpkg or
CMake quirk, a CI surprise, a working pattern for a tricky extension
feature — capture it here. A one-paragraph note under the relevant
section (`Common pitfalls` for gotchas, the appropriate topical
section for new patterns) is enough. The bar is *"would past-me have
saved an hour if this had been written down?"* If yes, write it.

## Repository layout

```
.
├── CMakeLists.txt              # explicit source list, find_package() per dep
├── Makefile                    # one-liner: include extension-ci-tools/.../duckdb_extension.Makefile
├── extension_config.cmake      # duckdb_extension_load(quack_oauth ...)
├── vcpkg.json                  # native deps (openssl today; more land per module)
├── src/
│   ├── quack_oauth_extension.cpp   # entry point: DUCKDB_CPP_EXTENSION_ENTRY → LoadInternal
│   └── include/
│       └── quack_oauth_extension.hpp
├── test/sql/                   # SQLLogicTest *.test files
├── duckdb/                     # submodule, pinned to v1.5.3
├── extension-ci-tools/         # submodule, pinned to v1.5.3
├── requirements.md             # functional spec
├── architecture.md             # arc42 design doc
└── docs/UPDATING.md            # how to bump the DuckDB target version
```

## Build

Always use ninja — every sibling extension does, and `make` without it
takes 2–3× longer:

```bash
export VCPKG_ROOT=/home/jr/.local/share/vcpkg          # or wherever your vcpkg lives
export VCPKG_TOOLCHAIN_PATH=$VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake
GEN=ninja make                # release build, usual dev loop
GEN=ninja make debug          # debug build (needed for ASAN, lldb, etc.)
```

Artifacts:

- `build/release/duckdb` — duckdb shell with the extension statically linked
- `build/release/test/unittest` — DuckDB unit test runner (also has the
  extension linked in; the way to run SQL tests)
- `build/release/extension/quack_oauth/quack_oauth.duckdb_extension` —
  the loadable binary that ships in distribution

First build is slow: DuckDB + the vcpkg openssl fetch. Incremental
builds are seconds with ninja. `make clean` is rarely needed — only
after editing `vcpkg.json` or pulling a new DuckDB submodule SHA.

## Tests

Two layers — see `docs/IMPLEMENTATION.md` section 2 for the policy.
**SQL tests are the source of truth**; Catch2 unit tests are
restricted to pure logic with no DuckDB linkage. Mocks are allowed
in Catch2 only, never in SQL tests.

```bash
make test                                                       # all SQL tests
./build/release/test/unittest test/sql/<feature>.test           # one SQL test
./build/release/test/unittest --test-dir . "[sql]"              # all sql group
make unit_test                                                  # Catch2 unit tests
./build/release/test/quack_oauth_unit_tests "[tracing][redact]" # one Catch2 tag
```

Static-linkage smoke (asserts only platform libs are dynamic deps):

```bash
make smoke_static
```

Test file header convention (copied from anofox-context):

```
# name: test/sql/<feature>.test
# description: <one line>
# group: [sql]

statement error
SELECT some_fn();
----
Catalog Error: Scalar Function with name some_fn does not exist!

require quack_oauth

query I
SELECT some_fn('x');
----
expected output
```

Always start the live-extension part of a test with `require quack_oauth`.
Use `statement error` *before* the require to assert the extension is
genuinely not loaded by default.

## vcpkg

We use vcpkg in manifest mode (`vcpkg.json`) with the overlay-ports
and overlay-triplets shipped inside `extension-ci-tools`. Today's only
declared dep is `openssl`.

To add a dependency (jwt-cpp, cpp-httplib, spdlog, … per
architecture.md §8):

1. Add the package name to `vcpkg.json` dependencies.
2. In `CMakeLists.txt`, add a `find_package(<name> CONFIG REQUIRED)`
   and link the import target in both `target_link_libraries()` calls
   (static **and** loadable).
3. `make clean && GEN=ninja make` — vcpkg will fetch and build the
   dep into `vcpkg_installed/`.

Header-only libs (jwt-cpp, spdlog when used as header-only) only need
an `include_directories(...)` or the corresponding interface target.

## Source layout conventions

Follow the anofox-tabular / anofox-context pattern:

- One paired `.cpp` + `.hpp` per module. Header in `src/include/`,
  implementation in `src/`. No subdirectories under `src/` until a
  module grows past ~5 files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DataZooDE/quack-oauth](https://github.com/DataZooDE/quack-oauth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
