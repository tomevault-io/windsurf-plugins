---
trigger: always_on
description: Guidance for working on `nanobench` — a single-header C++11..20 microbenchmark library
---

# CLAUDE.md

Guidance for working on `nanobench` — a single-header C++11..20 microbenchmark library
(`ankerl::nanobench::Bench`).

The whole library is `src/include/nanobench.h` (~3.5k lines). Everything below
`#if defined(ANKERL_NANOBENCH_IMPLEMENT)` is the implementation and is compiled into exactly one TU
(`src/test/app/nanobench.cpp`). Tests, examples and tutorials are in `src/test/` and link into a
single doctest binary, `nb`.

## Build & test

```sh
cmake -S . -B build -GNinja -DCMAKE_BUILD_TYPE=Release
ninja -C build
./build/nb                     # all tests;  -ltc lists them,  -tc=<name> runs one,  -s shows detail
```

Options: `-DNB_cxx_standard=17` (default 11), `-DNB_sanitizer=ON` (gcc and clang both supported).

**Build gotchas**

- clang builds use `-Weverything -Werror`, which opts in to warnings that don't exist yet, so a new
  clang can break the build. Version-guarding each `-Wno-` doesn't work — AppleClang's version
  numbers don't map to upstream clang's — so `src/cmake/CMakeLists.txt` uses
  `-Wno-unknown-warning-option` plus a list. Put only nanobench's *own* warnings in that list:
  warnings from the vendored `doctest.h` are handled by including `src/test` as a `SYSTEM` directory,
  which suppresses them as a class instead of blinding `nanobench.h` to them too.
- clang-tidy is *not* part of the build (it broke every clang build whenever it gained a check,
  issue #108) — the `lint` CI job runs it pinned. Locally: `-DCMAKE_CXX_CLANG_TIDY=clang-tidy-18`.
- Running `./nb` writes example artifacts (`*.json`, `mustache.*`, `always_the_same.html`, …) into
  the *current* directory. Run it from the build dir, and check `git status` before `git add -A`.
- `unit_templates` compares the built-in templates against `src/docs/_generated/*` using a path
  derived from `__FILE__`, so it only passes when `__FILE__` is absolute (cmake) or cwd is the repo
  root. A failure there after a manual compile is an artifact, not a real regression.
- A test that parses the markdown table must give its `Bench` its own `title()`. The header is only
  written when the table's shape changes, and that state lives on the output stream, so a second test
  configuring the table identically gets **no header at all** — after which a helper that indexes the
  header row crashes instead of failing. Keep such helpers total for the same reason.
- Don't name anything in `src/test/` after a `<cmath>` function. libc++ puts `::fma` in scope, so a
  local `fma` template joins it in one overload set and `fma<float>` stops resolving — that broke
  the libc++ leg until `tutorial_context.cpp`'s helper became `fma_bench`.

## Landing a change

Every change reaches `master` through a pull request that is green — one-line fixes, docs edits and
lint reformats included. This is enforced rather than agreed: `master` requires the `CI green` status
check with `enforce_admins` on, so a direct push is rejected with `GH006: Protected branch update
failed`. Rebase merge is the only method the repository allows, so `gh pr merge <n> --rebase` is the
call. Merging someone else's PR is fine once its checks pass.

The rule is here because local verification cannot stand in for the matrix, however thorough it
looks. It was introduced after a commit pushed straight to `master` turned eleven clang and macOS
legs red: a new test whose empty-capture lambda instantiated `SetupRunner` for the first time tripped
`-Wpadded`, and the pre-push check had only run clang over the header — where that template never
gets instantiated at all.

`CI green` is a gate job at the end of `main.yml` that `needs:` every other job in the workflow, so
protection requires one check instead of thirty job names that silently stop being required as legs
get renamed or added. Two parts of it are load-bearing: `if: always()`, because a job whose
dependency failed is *skipped* rather than failed, and the explicit result test, because GitHub
counts a skipped required check as **satisfied** — without both, the gate would go green exactly when
the matrix did not. `lint-ci-gate.py` fails the build if a job is missing from its `needs:`, if an
entry is stale, or if `if: always()` disappears.

Old PRs still carry red and green checks from Travis, Cirrus and AppVeyor. All three are dead for
this repository and their results mean nothing — read the GitHub Actions checks instead.

## Verifying locally

Nothing below replaces the PR: it is how you arrive with a change that has a chance of being green.
`.github/workflows/main.yml` builds every leg the same way, so any of them reproduces locally:

```sh
CXX=<compiler> cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DNB_cxx_standard=<std> <cmake_args>
cmake --build build --parallel 4 && ./build/nb        # run from the repo root
```

It covers gcc/clang × C++11..20, 32 bit, libc++, sanitizers, ARM64, macOS, MSVC, clang-cl and
MinGW, plus a `lint` job (pinned clang-format-18 / clang-tidy-18) and a CMake consumer job. What it
cannot cover is the pre-gcc-5 half of `src/scripts/all.sh`, which is what that script is still for.

Three legs exist because a bug hid for years behind everything else being alike:

- `musl libc (Alpine, gcc)` — glibc declares `ioctl()`'s request parameter as `unsigned long`, musl

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [martinus/nanobench](https://github.com/martinus/nanobench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
