---
trigger: always_on
description: - Prioritize non-verbosity and information density.
---

# KomaMRI.jl

## Style
- Prioritize non-verbosity and information density.
- Prioritize elegance and simplicity: concise idiomatic Julia, simple data expressions over stateful loops, no abstract fields, no speculative abstractions.
- Type function arguments only when dispatch needs them.
- Do not use `isa`/`typeof` type checks for behavior that can be expressed with dispatch or small trait functions.
- Use `!` only for functions that mutate an argument. Internal mutating helpers should return `nothing`; value-returning helpers should not use `!`.
- Use low-level constructors like `new{typeof(...)}` only in the minimal inner-constructor boundary that actually needs them.
- Add helper functions only when reused or semantically justified.
- Reuse existing APIs before adding private wrappers.
- Add docstrings only for public-facing functions.
- Prefer domain names over implementation-mechanic names and explanatory comments. Add comments only when they add information.
- Avoid magic numbers. Name domain constants and test fixtures when the value carries meaning.
- Make semantic phases visible in code structure.
- Be terse. Assume expert user.
- Prefer commands/diffs over long explanations.
- Keep diffs tight. Do not reformat or clean up unrelated code.
- Reduce lines by simplifying design, not by compressing code.

## Git
- If the user gives an existing branch/worktree, use it after verifying it is the intended one.
- If starting feature/PR work yourself, create a unique branch from latest `origin/master` in a separate worktree; keep the main checkout on `master`.
- If the active checkout is detached, create a unique local branch there before editing.
- Fresh worktree pattern: `git fetch origin && git worktree prune && git worktree add -b codex/<task> ../KomaMRI.jl-<task> origin/master`.
- If a branch/path exists, choose a new name. Do not force-remove unless asked.
- Never push unless the latest user message explicitly asks. Never push to `master`.
- When creating pull requests, open them ready for review unless explicitly asked for a draft PR.
- Do not prefix PR titles with `[codex]` or other author/tool tags unless explicitly requested.
- No destructive Git commands without explicit approval.
- Keep generated scratch artifacts under `.tmp/`; never stage or push `.tmp/` contents.

## Repo
- Monorepo packages: root `KomaMRI`, plus `KomaMRIBase`, `KomaMRICore`, `KomaMRIFiles`, `KomaMRIPlots`.
- `KomaMRICore`, `KomaMRIFiles`, and `KomaMRIPlots` depend on `KomaMRIBase`; the root package depends on those three.
- Each package has its own `Project.toml`; tests use `*/test/Project.toml`; docs use `docs/Project.toml`.
- Root `[workspace]` includes `test`, `docs`, `benchmarks`, and all subpackages.
- Do not edit `Manifest.toml` directly or commit manifest churn unless explicitly asked.

## Julia
- Use the relevant project: `julia --project=<path>` and `Pkg.activate(...)`; never use the global env by accident.
- First action for Julia work: use or start the persistent Julia REPL; do not run Julia through `exec_command` unless isolation is technically required and stated first.
- Use one persistent Julia REPL/session started with `--threads=auto` and Revise for all Julia work, including diagnostics, plotting, scratch scripts, examples, and tests. Do not run `julia -e`, `julia script.jl`, or package tests in fresh shell processes while a threaded REPL is available; send commands to the session. If code is too large to paste safely, write it under `.tmp/` and run `include("...")` from the existing REPL. Use a fresh process only when isolation is technically required or the user asks. Restart only if absent/crashed, incorrectly threaded, or the user asks. Keep it open.
- On Julia 1.12+, Revise can handle struct redefinitions; do not restart only because a struct changed.
- Prefer workspace setup: activate root or the child project and `Pkg.instantiate()`. Use explicit `Pkg.develop(path=...)` only to reproduce CI or older Julia 1.10 wiring.
- Change dependencies/compat with Pkg APIs, not casual `Project.toml` edits.
- Prefer command-line arguments for script options. Use environment variables only for actual environment/CI/backend semantics such as CPU/GPU/CUDA/Metal selection.
- For performance work: profile first, benchmark with `BenchmarkTools.@benchmark` and interpolated inputs, then optimize the actual hotspot. Never report one-off `@elapsed`/wall-clock timings as benchmark results. Warm the code first; for GPU benchmarks, synchronize the backend (for example `CUDA.synchronize()`, `Metal.synchronize()`) inside the measured function or immediately after the kernel work.
- Never write raw `seq += ...` in examples or generated code; use `@addblock`.
- Never call `build_*` only to extract events or duration and then rebuild the same block. Use `make_*` for custom blocks, copy a built block when preserving its block semantics, or append the built sequence/block directly.

## Python
- Use `uv` for reproducible Python environments. Do not use bare `pip`.

## Testing
- In the persistent Julia session, run tests from the correct project with `Pkg.test(...)`; use the narrowest package first.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JuliaHealth/KomaMRI.jl](https://github.com/JuliaHealth/KomaMRI.jl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
