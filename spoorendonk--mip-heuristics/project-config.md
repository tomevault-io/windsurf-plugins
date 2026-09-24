---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Navigation: LSP → narrow Grep → sliced Read.

# Standards

## Communication Style

Be terse. No preamble. No filler.

## Code Navigation

Prefer narrow queries over full-file reads:

1. **LSP** for symbol questions — `goToDefinition`, `hover`, `documentSymbol`, `workspaceSymbol`. Use before `Read`.
2. **Grep** with `-n` and a small `head_limit` (start at 20); raise only if inconclusive.
3. **Read** with `offset`/`limit` for a slice around the hit. Full-file `Read` is fine under ~200 lines or when structure matters.

Know the symbol → LSP. Know a string, not its location → Grep. Full-file Read is the last mile. This is a preference, not a prohibition: shelling out to `grep`/`rg` is fine when the built-in can't do the job (filtering a pipe like `git log | grep`, or a session without the `Grep` tool). What matters is bounding output, not which binary produces it.

Setup: install `clangd-lsp@claude-plugins-official` plus `clangd` (`apt install clangd`, or from LLVM). `.clangd` points at `build/compile_commands.json`, produced by `CMAKE_EXPORT_COMPILE_COMMANDS ON`.

## C++

- Target C++23. Use modern features (`std::expected`, concepts, ranges, `constexpr`).
- Use `#pragma once`. Minimize header includes; forward-declare where possible.
- **Formatting** is Google, via `.clang-format`. **Naming is not Google** — `.clang-tidy` enforces what this codebase actually does:

  | Kind | Style | Example |
  |---|---|---|
  | Functions (free and member) | `lower_case` | `run_sequential`, `charge_presolve` |
  | Locals, parameters, `const` locals | `lower_case` | `per_worker`, `max_effort` |
  | Private members | **trailing** `_` | `FjWorker::start_`, `seed_` |
  | File-scope / static / class / `constexpr` constants | `k` + `CamelCase` | `kMaxStaleRounds`, `kNumLpArms` |
  | Classes | `CamelCase` | `ScyllaWorker`, `EffortLedger` |
  | Namespaces | `lower_case` | `heuristics`, `fpr` |
  | Files | `snake_case.h` / `.cpp` | |

  `ConstantMemberCase` is deliberately unset: it outranks `PrivateMemberSuffix`, so setting it would reject a `const` private member spelled with this project's `_` suffix. The `.clang-tidy` comments carry the reasoning for each narrowing — read them before changing one.

## Complexity

When a complexity warning fires, don't extract methods mechanically. Ask what the independent responsibilities are and split along those boundaries. If the function is genuinely complex because the domain is, add a comment explaining why and suppress the warning.

## CMake

- `set(CMAKE_EXPORT_COMPILE_COMMANDS ON)` for clang-tidy.
- Use FetchContent for dependencies.
- A single root `CMakeLists.txt`; per-directory files would only add indirection.

## Testing (Catch2 v3)

- `TEST_CASE("name", "[tag]")` with `[tag]` filters, in `tests/`. Not GoogleTest.
- Instances come from HiGHS's own `check/instances/` (path injected as the `INSTANCES_DIR` compile definition).
- `ctest --progress` collapses the running list and `CMAKE_INSTALL_MESSAGE=LAZY` suppresses install chatter. Don't remove these.
- **`[serial]` is a reserved tag**, and it is the *last* resort for a timing-dependent test. `CMakeLists.txt` registers the tagged cases through their own `catch_discover_tests` call carrying `TEST_SPEC "[serial]"` and `PROPERTIES RUN_SERIAL TRUE`, beside a second call taking `TEST_SPEC "~[serial]"` for everything else; the two specs must partition the suite exactly (verify with `ctest -N | wc -l` and a `uniq -d` over the names). Give each call its own `TEST_LIST` name.

  Order of preference when a test flakes under `ctest -j$(nproc)`: **(1) take the clock out of the fixture** — e.g. HiGHS's free-format MPS reader spends `time_limit` as its own *parse* budget, so write the option *after* `readModel`, as `build_bare_mipsolver` in `tests/test_common.h` does; **(2) assert the mechanism** — a one-sided effort bound like `effort < attempt_cap` only gets easier under starvation, so it needs nothing; **(3) only then tag it**, for a wait nothing in the code can bound. **The load-safety of an effort bound is a property of its *shape*, not of the quantity it names**: the *differential* form `effort(T) < effort(5T)` that `tests/test_deadline.cpp`'s `require_clock_bound` uses is two-sided and fails if the longer run is descheduled harder, so it is a (3) case and carries `[serial]` even though it is an effort assertion.

  **Never widen a constant** to stop a flake — a tuned threshold is the same test with a longer fuse. `RUN_SERIAL`, not `RESOURCE_LOCK`: these need an unloaded machine, not mutual exclusion, and neither excludes load from outside ctest, which is why (1) and (2) beat (3) whenever available.

## Development Workflow

```
plan (non-trivial) → implement → test → push to main
```

Nothing formats on save — there are no Claude Code hooks. `clang-format` runs at commit time: `pre-commit` formats the staged C++, applies safe `clang-tidy` fixes, and re-stages the result, so what you commit is canonical even though the file you just edited is not. Don't hand-tune formatting — let the hook normalize it, or run it yourself:

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spoorendonk/mip-heuristics](https://github.com/spoorendonk/mip-heuristics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
