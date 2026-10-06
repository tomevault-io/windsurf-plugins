---
trigger: always_on
description: Guidance for AI coding agents working in `EVerest`, the mono-repository formerly named
---

# EVerest: Agent Guidelines

Guidance for AI coding agents working in `EVerest`, the mono-repository formerly named
`everest-core`: runtime manager, modules, shared libraries, interface definitions, test
infrastructure. C++ primary, plus Python, Rust and JavaScript. CMake, any generator;
CI also builds with Bazel.

Libraries that once lived in separate `lib*` repositories are now in-tree under
`lib/everest/` and are edited in place; there is no upstream repository to mirror them
to. A few EVerest components are still external dependencies in `dependencies.yaml`,
notably the Python Josev stack (`ext-switchev-iso15118`).

Contributor policy (licensing, DCO, review, and the project's position on AI-generated
contributions) is in `docs/source/project/contributing.rst`. This file covers mechanics
only: build, run, test, code generation, and invariants.

## Documentation

Prefer project docs over your own assumptions, and cite them. When docs and the tree
disagree, the tree wins: manifests, `interfaces/*.yaml` and code describe current
behavior, docs describe intent. Say so when you find the two diverging.

Docs are rendered at <https://everest.github.io>, starting with the
[Getting Started Guides](https://everest.github.io/nightly/how-to-guides/getting-started/index.html).
Sphinx sources are under `docs/source/`: `explanation/` for concepts, `how-to-guides/`
for recipes. Per-module docs are at `modules/<Category>/<Module>/docs/index.rst`.

Docs are updated in the same change as the code they describe. After changing a
module's public behavior (interface or type YAML, config keys, state machines, error
semantics), check the module's `docs/index.rst` and any `docs/source/` page describing
that behavior, and fix what went stale. Libraries document themselves in
`lib/everest/<lib>/README.md` and their own doc trees (`docs/` usually, `doc/` in
libocpp); internal library changes count too when they touch documented behavior. A
change to build, run, test or codegen mechanics means updating this file as well.

## Build

`make build`, `make test` and `make lint` do not exist here.

```bash
cmake -S . -B build \
  -DCMAKE_INSTALL_PREFIX=./build/dist \
  -DCMAKE_BUILD_TYPE=Debug \
  -DCMAKE_EXPORT_COMPILE_COMMANDS=ON \
  -DBUILD_TESTING=ON \
  -DEVEREST_ENABLE_COMPILE_WARNINGS=ON
cmake --build build --target install
```

Pass `-j` yourself, since generator defaults are either the whole machine or a single
job. When several agents share a host, cap the total to the core count.

Dependencies are pinned in `dependencies.yaml` and fetched by edm at configure time;
`-DDISABLE_EDM=ON` uses system packages. `ISO15118_2_GENERATE_AND_INSTALL_CERTIFICATES`
defaults to ON, so development certificates need no extra flag.

To iterate faster, build one module (`cmake --build build --target OCPP201`) or skip
heavy ones with `-DEVEREST_EXCLUDE_MODULES="EvseSlac;EvseV2G;IsoMux"`, quoting the
list. `cmake -LH build` lists all options; the two easiest to miss are
`EVEREST_ENABLE_COVERAGE` and `CMAKE_RUN_CLANG_TIDY`, both OFF.

## Running

Generated run scripts are the simplest entry point:

```bash
ls build/run-scripts/                 # every generated configuration
build/run-scripts/run-sil.sh          # AC software-in-the-loop
```

Each wraps the manager with the environment it needs (template
`cmake/assets/run_template.sh.in`). By hand, set the same three things:

```bash
LD_LIBRARY_PATH=<prefix>/<libdir>:$LD_LIBRARY_PATH \
  ./build/dist/bin/manager --prefix <prefix> --config <full path to config yaml>
```

`<libdir>` is `lib` or `lib64`, following `CMAKE_INSTALL_LIBDIR`. `--config` also takes
an extensionless name, looked up in the installed config directory. `--check` validates
a config and exits 0 on success.

Logging is one Boost.Log config for the whole system, not per module: the manager reads
`<prefix>/etc/everest/default_logging.cfg` (`cmake/assets/logging.ini` renamed at
install; `--log_config` overrides). Per-module verbosity is a filter in that file:

```ini
Filter="(%Process% contains OCPP201 and %Severity% >= DEBG) or %Severity% >= INFO"
```

Without the `or` clause, other modules go silent. Severities: DEBG, INFO, WARN, ERRO,
CRIT.

## Testing

```bash
cd build && ctest --output-on-failure
cd build && ctest -N                  # list registered names before filtering
cd build && ctest -R <regex>
```

`ctest -R` matches **registered test names, not build targets**, and the two often differ
in case and spelling. List with `ctest -N` first. Coverage needs
`-DEVEREST_ENABLE_COVERAGE=ON`; the `everest-core_create_coverage` target writes
`build/everest-core_create_coverage/index.html`.

CI runs the unit tests in parallel (`ctest -j "$(nproc)"` in
`.ci/build-kit/scripts/run_unit_tests.sh`), so a test must not depend on running alone.
Prefer port 0 and per-test temporary directories. Tests that share a fixed resource (a
port, a file path, a PKI or database directory) get a common `RESOURCE_LOCK`, as the libtls
and io TLS tests do with `tls_pki`; a test that must not run next to any other gets
`RUN_SERIAL`, as the iso15118 tests on the fixed port 50000 do. With
`catch_discover_tests` or `gtest_discover_tests`, set them through `PROPERTIES` so they

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EVerest/EVerest](https://github.com/EVerest/EVerest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
