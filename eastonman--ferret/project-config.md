---
trigger: always_on
description: Operational guide for both human contributors and agentic workers
---

# AGENTS.md

Operational guide for both human contributors and agentic workers
(LLM coding assistants). The discursive docs live in
[`docs/`](docs/); this file is the short, command-oriented checklist
of what to run, in what order, before opening a PR.

## What ferret is

A JIT-driven microbenchmark framework for probing CPU frontend
microarchitectural structures (BTB, RAS, BPU, decoded-uop cache,
ITLB). C++20 core under `src/`, benchmarks under `benchmarks/`, a
Python plotting CLI under `scripts/`. See [`README.md`](README.md)
and [`docs/architecture.md`](docs/architecture.md) for the full
mental model.

## TL;DR pre-PR sequence

Inside `nix develop` (or with the equivalent tools on `PATH`):

```sh
# format C++ + Python + CMake + Markdown in place.
./scripts/format.sh

# generate compile_commands.json for lint.
cmake -S . -B build -GNinja -DCMAKE_EXPORT_COMPILE_COMMANDS=ON

# build all targets.
cmake --build build

# run C++ tests.
ctest --test-dir build --output-on-failure

# run Python tests (skips the `integration` marker).
./scripts/test_py.sh

# verify everything CI verifies — this is the gate.
./scripts/lint.sh
```

All six must exit zero before opening a PR. There are **no
pre-commit hooks** — `lint.sh` is enforced only in CI, so running
it locally is the only way to catch a CI failure before pushing.

## Build directory rule

**Always use `build/`.** Several `build-*/` directories may exist
locally from past experiments (`build-codereview/`, `build-lint/`,
`build-x86-linux-asanubsan/`, …); they are not canonical and not
tracked by git. `scripts/lint.sh` reads `build/compile_commands.json`.
If you must keep a side-build (e.g., a sanitizer tree), use a
distinct directory and leave `build/` as the lint-friendly tree.

## C++ workflow specifics

- **Warnings are errors.** `FERRET_WERROR=ON` is the CMake default
  and applies to ferret's own targets (not vendored deps). Do not
  commit a build that required `-DFERRET_WERROR=OFF`.
- **clang-tidy is hard.** `.clang-tidy` has `WarningsAsErrors: '*'`,
  so any diagnostic from the configured `bugprone-*`, `performance-*`,
  `readability-*`, `modernize-*` checks fails CI. `lint.sh` skips
  files under `tests/` and under `_deps/`; everything else under
  `src/`, `include/`, `benchmarks/` is linted at the compile-commands
  level.
- **Sanitizer trees are separate.** ASan and TSan need independent
  instrumented builds — configure them into distinct directories
  (`build-asan/`, `build-tsan/`). See
  [`docs/build.md`](docs/build.md) for the env-var recipe that
  matches CI.
- **Per-platform sources.** `CMakeLists.txt` picks
  `src/timing/{x86_64,aarch64}.cpp`, `src/pinning/{linux,macos}.cpp`,
  and `src/padding/{x86_64,aarch64}.cpp` at configure time. If you
  touch one arch/OS path, build the matching one too — CI runs both.

## Python workflow specifics

- `ruff>=0.14` is required (`pyproject.toml` pins `required-version`);
  older ruff hard-errors on startup. The Nix dev shell provides a
  pinned version.
- `tests/python/` is the suite; `conftest.py` puts `scripts/` on
  `sys.path` so `import ferret_plot` works without an install.
- The `integration` marker gates tests that spawn Chrome via kaleido
  (`tests/python/test_integration_export.py`). `scripts/test_py.sh`
  excludes them with `-m "not integration"`; CI runs the integration
  suite separately on Linux + macOS with Chrome on `PATH`.

## Running a single test

```sh
ctest --test-dir build -R test_timing --output-on-failure
./build/tests/test_timing --gtest_filter='Timing.TicksPerNsIsPositive'

python3 -m pytest tests/python/test_surface.py
python3 -m pytest tests/python/test_surface.py::test_some_case -v
python3 -m pytest tests/python -m integration -v   # requires Chrome
```

`ctest` only knows about C++ targets — Python tests are not
registered with it.

## Commit messages

`type(scope): subject` (Conventional-Commits-ish). Scope is
optional; subject is lowercase, imperative, no trailing period.

Types in use: `feat`, `fix`, `refactor`, `style`, `perf`, `build`,
`ci`, `test`, `docs`, `lint`. Examples from `git log`:

```text
refactor(surface): decompose make_figure and name tunables
perf(plot): lazy-import kinds and clean up browser staging files
build: tighten sljit visibility and share benchmark objects
fix(plot): export surface png via chromium webgl fallback
lint: silence ruff on intentional lazy imports and wide signatures
```

Forbidden in commit messages, PR titles, PR bodies, and code
comments:

- AI tool names (Codex, Claude, Grok, Gemini, …) — including
  `Co-Authored-By:` footers.
- Process narration ("FIXED", "Step 3", "Week 2", "Phase 1",
  "AC-x"). Write what the change _is_, not how the work
  progressed.

## Branches and PRs

Local branches follow `type/short-description`, matching the commit
type vocabulary: `feat/branch-history-footprint`,
`fix/surface-png-webgl-export`, `refactor/code-review-cleanup`,
`ci/workflow-hardening`.

PRs target `main`. CI must be green:

| Workflow         | What it gates                                                                                                           |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eastonman/ferret](https://github.com/eastonman/ferret) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
