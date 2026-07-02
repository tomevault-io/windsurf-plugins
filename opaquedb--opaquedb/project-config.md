---
trigger: always_on
description: Guidance for working in this repository. Keep it current: update it when
---

# CLAUDE.md

Guidance for working in this repository. Keep it current: update it when
layering, build/release commands, conventions, or step status change.

## What this is

OpaqueDB is a distributed database that answers SQL-like queries over
homomorphically encrypted data using Microsoft SEAL (BFV). It is a computational
PIR scheme. Privacy holds against a semi-honest operator with no non-collusion
assumption (single trust domain), resting on Ring-LWE. The deployed unit is a
sharded cluster of identical nodes. A cluster holds many databases, each holding
many tables; a query routes to the table named in its SQL within the database
the request selects (default "default").

## Build, test, lint

The dev container has cmake, ninja, vcpkg (`$VCPKG_ROOT=/opt/vcpkg`), and the
lint tools. All dependencies flow through vcpkg in manifest mode. The `Makefile`
is the single soCommon tasks run through the `Makefile`, which is the single source of truth for
build, test, lint, and packaging commands (CI invokes the same targets):

```sh
export VCPKG_ROOT=/opt/vcpkg   # provided by the dev container
make configure                 # first run builds dependencies via vcpkg
make build
make test
```

Run `make help` to list every target. Useful ones: `make lint` (clang-format and
clang-tidy), `make package` (release `.deb` and `.tar.gz`), and `PRESET=release`
on `configure`/`build` for an optimized build.

The dev container in `.devcontainer/` provides the C++20 toolchain and vcpkg.
The first configure is slow because vcpkg builds dependencies from source; later
builds use the binary cache.urce of truth for common tasks; CI and the docs call the same
targets. `make help` lists them.

```
make configure   # cmake --preset dev (Debug, -Werror); first run builds deps
make build       # cmake --build --preset dev
make test        # ctest --preset $(PRESET) (dev by default)
make test-fast   # configure+build+test the release preset (fast SEAL)
make coverage    # instrumented run -> build/coverage/coverage.html
make hooks       # enable .githooks (pre-commit clang-formats staged C++)
make lint        # clang-format check + clang-tidy (CI runs format only)
make package     # CPack .deb (full) + binary-only .tar.gz
```

Formatting on commit: the tracked `.githooks/pre-commit` runs clang-format over
the staged C/C++ files and restages them, so a commit always satisfies CI's
`make format-check`. Enable it once per clone with `make hooks` (sets
`core.hooksPath .githooks`); it no-ops if clang-format is absent.

Test speed: Debug links debug-SEAL, so the heavy matcher tests take 50-120s
each; the full Debug suite is over ten minutes. The release preset links the
optimized SEAL and runs the same suite in ~35s, the fast tier (`ctest -LE slow`)
in under a second. Prefer `make test PRESET=release` (or `make test-fast`) for
the inner loop; keep a Debug run for the `-Werror` / `-O0` checks. The SEAL-heavy
tests (crypto, reference backend, server, distributed) carry the `slow` ctest
label: `ctest -LE slow` skips them, `ctest -L slow` runs only them.

Coverage: `make coverage` uses the `coverage` preset (optimized deps so SEAL is
fast, first-party code at `-O0` with `--coverage` so line attribution is exact)
and renders a gcovr report. The vcpkg deps are never instrumented because they
do not link the `opaquedb_compile_options` interface target that carries the
flags. Baseline at the time this was wired: 63% lines, 79% functions, 31%
branches over `src/`; the CLI commands, `log`, and `real_etcd_client` are near
0% (no harness yet) and branch coverage is low because error/validation paths
are largely untested. The `coverage` preset passes the full suite from a clean
build; an incremental reconfigure that relinks libraries without recompiling a
test binary can leave it stale and trip spurious failures, so run `make coverage`
against a clean `build/coverage` (rm it first if you changed flags).

Override the preset with `PRESET=release` (e.g. `make build PRESET=release`). On
a fresh build dir use the gitignored `dev-local` preset (it sets the buildtrees
relocation, below). The raw `cmake --preset dev` / `cmake --build` / `ctest`
commands still work; the Makefile just wraps them.

Performance: `dev` is Debug (`-O0`), where SEAL arithmetic is ~30x slower. At
poly 16384 a small query is tens of seconds in Debug versus well under a second
warm in Release; a cold run is dominated by one-time key setup (the ~125 MB
Galois set), not the matching. Always measure real workloads with
`make build PRESET=release`; the binary is `build/release/opaquedb`.

### Versioning and release

Git-flow (AVH) drives branches: `main` is production, `develop` is integration,
work merges through `release/*` and `hotfix/*`; tags are `vX.Y.Z`. The version
lives only in CMake `project(VERSION)` (checked against `vcpkg.json` at configure
time) and flows to `OPAQUEDB_VERSION`, `--version`, and CPack. Finishing a
release (`git flow release finish X.Y.Z`) tags `vX.Y.Z` on `main`. `make
package` produces two artifacts, both named Debian-style
(`opaquedb_<version>_<arch>`): the `.deb` (CPack, full install tree: binary,
systemd unit, example config) and a binary-only `.tar.gz` (the `tarball` CMake

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opaquedb/opaquedb](https://github.com/opaquedb/opaquedb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
