---
trigger: always_on
description: Coverage-guided binary fuzzer: ASAN/MSAN/TSAN/UBSAN detection, dictionary mutations,
---

# AGENTS.md — fuzzer-tool

Coverage-guided binary fuzzer: ASAN/MSAN/TSAN/UBSAN detection, dictionary mutations,
Markov chain generation, Monte Carlo optimization, format-aware grammar mutations, and
state persistence. CLI tool that fuzzes arbitrary binaries (see `fuzzer-tool --help`).

The fuzzer executes attacker-controlled input against instrumented targets and parses
the targets' own binaries — any bug in this tool's parsing/process code is a bug in the
fuzzer, not just the target.

## References (read on demand)

| File | Open when |
|------|-----------|
| `docs/refs/bug-classes.md` | Touching process/signal handling, timeouts, ptrace, concurrency, resource cleanup, hashing/identity, caching, ELF/low-level parsing, numeric edge cases, state persistence, dispatch tables, error swallowing, .so symbol visibility, widely-used return-value APIs, or test mocks. Also carries the regression-testing rules. |
| `docs/refs/architecture.md` | Working inside coverage/SHM internals, the AFL shim, `--no-shm`/`--deep-coverage`, the Elo meta-scheduler, or state persistence (`state.json` / `edge_tracker.json` / `markov.json`). |

## Hard Rules

0. Always make surgical changes.
1. **Always follow existing conventions.** Before adding anything — a target, a script, a scheduler, a test fixture — find the closest existing example and match it: directory layout, file naming, function shape, flag names, error handling, comment style. Read the surrounding code first; do not invent a parallel way of doing something the repo already does. Concretely: vendored library sources go in `vendor/<lib>/` (gitignored) fetched by a `tools/vendor_<lib>.sh` script — never committed and never under `targets/`; new fuzz targets are wired into `tools/build_targets.sh` rather than built by hand; new schedulers register in `_OPERATOR_STRATEGY_NAMES` and follow the `select_op`/`record`/`bandit_stats` interface. If a convention appears wrong, fix it in one place for everything rather than working around it locally, and say so.
2. Never bypass the pre-commit hooks (`--no-verify`). Fix the warnings, then recommit.
3. Always fix impactguard breaking changes.
4. Always use clang, never gcc (build scripts prefer clang automatically).
5. Stop suggesting `use_direct_lite = False` to work around ASAN in `direct_lite` mode — debug the root cause instead.
6. Never commit binary files or corpus directories — build targets from source, keep corpus data local.
7. Before deleting code, find where it should be wired first; if not found, clean up. When removing code, stay strictly within the scope of the removal — do not remove unrelated code.
8. Do not nuke the repo.
9. All fuzz targets: compile with ASAN (`-fsanitize=address`) and AFL edge coverage via `afl_shim.c` (`-include src/fuzzer_tool/adapters/afl_shim.c`). Pre-compile library sources as `.o` files; link the shim only into the target wrapper.
10. Always create TODOs. Always commit and push after finishing a task.
11. Update `docs/DEEP_DIVE.md` with new features (the comprehensive reference). Update `README.md` only when adding or changing high-level capabilities visible in the quick-start or feature overview.
12. Op mutators have a single source of truth: `src/fuzzer_tool/core/operator_registry.py`'s `REGISTRY`. Register new operators there only — the dispatch table (`build_dispatch`), the per-input op list (`build_ops`), scheduler arming (`_register_arms`), and `OPERATOR_CATEGORIES` all derive from it. Never add operator names to the legacy `MUTATIONS`/`FORMAT_MUTATIONS`/`DICT_MUTATIONS` lists or hand-edit `OPERATOR_CATEGORIES`; schedulers discover ops through the services layer and never hardcode op lists.
13. Only run the full pytest suite if a file in the codebase was modified.
14. Always after developing a new function and verify its correctness try to vectorize it after the fact, keep the fastest version.
15. Always use the existing pickle machinery, avoid json.
16. For random always use the prng in `src/fuzzer_tool/core/rand_pool.py`.
17. Do not create artifacts in the source codebase dir.
18. Always create corpus on ~/.
19. Always make sure when creating a new functionality that is wired-up where needed.
20. try except pass is a bad pattern.
21. Always excersice higiene: every test must clean up their mess.
22. If you solved the halting problem you are allowed to run tests that the user is saying they are hanging otherwise read the code to see what it does.
23. For every new functionality always add one falsification test and one adversarial test.
24. Always use subagents for multiple file exploring or long tasks.
25. Without any question when the user instructs you to git and commit now, you obey, create the commit message, commit and push. no questions asked and no deliberation.
26. When writing something intended for human consumption, (comment, commit message, reply to prompt) use as few words as possible. Pick every word meticulously to reduce the volume to a strict minimum. Be down to the point. Less is more.
27. Avoid superlatives and praise. Stop telling me I am absolutely right. Give me the cold hard truth.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daedalus/fuzzer](https://github.com/daedalus/fuzzer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
