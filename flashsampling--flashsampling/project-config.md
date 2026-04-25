---
trigger: always_on
description: - the speed test runner is in `benchmarking/speed_test.py`, and the triton benchmark runner is in `benchmarking/triton_benchmark.py`. There are useful commands to run benchs in `./benchmarking/Makefile`.
---

# AGENTS.md

- the speed test runner is in `benchmarking/speed_test.py`, and the triton benchmark runner is in `benchmarking/triton_benchmark.py`. There are useful commands to run benchs in `./benchmarking/Makefile`.
- equivalent commands for modal can be found in `./Makefile`.
- The shared `Args` class for both speed_test and triton_benchmark lives in `triton_benchmark_lib.py`. speed_test's `CliArgs` overrides defaults (`case="small"`, `n_hidden_states=1`).
- to plot all plots use `make plot-all`.
- to test the distributed code works use `make modal-pytest-distributed` (requires >= 2 GPUs with NVLink for symmetric memory). `make pytest-distributed` also works but auto-skips on single-GPU machines.
- when benchmarking many combinations, don't run the bench in parallel, since they will contend for the same resources. Instead launch them sequentially. On modal, you can launch benchmarks in parallel, since each job should get its own resources. When launching many parallel Modal benchmarks with an empty Triton autotune cache, consider running a single warmup job first to populate the cache on the volume. Otherwise every parallel job will autotune independently, wasting GPU time and risking inconsistent config selection.
- available CLIs: hugging face, brev, github
- The blog post is in `~/code/tomasruizt.github.io/tomas-blog/posts/07_fused-mm-sample/index.qmd`.
- The paper is in `~/code/papers/flashsampling-paper/`.
- The FMMS Triton kernel is in `src/fused_mm_sampling/core.py`.
- Generally speaking, do imports at the top of the file, not inside functions.
- Do no speculate blindly about why code is slow. Causal statements need to be backed by empirical evidence. Choose appropriate language to hedge, e.g. "Possibly", "Potentially", and point out what data would let us clear the uncertainty and make confident claims.

Development notes and lessons learned while building this project.

**Meta-rule: Continuously update this file.** After every task, write new insights, patterns, and lessons learned into this file. Proactively review and update outdated information — if a timeout was changed, a cache strategy was revised, or a workaround is no longer needed, update the relevant section. This file is the project's living knowledge base.

## Code style

- **Top-down structure**: Define high-level functions first, helpers below. A reader should encounter the main logic before the details it delegates to. Helper functions go **after** the function that calls them, not before.
- **Never introduce GPU-CPU synchronizations.** Operations like `tensor.item()`, `float(tensor)`, `tensor.cpu()`, or `print(tensor)` on CUDA tensors force the CPU to wait for all pending GPU work to finish, destroying pipeline parallelism. Pass scalar values as 0-d CUDA tensors instead of extracting Python floats. Both the Triton kernel (`tl.load(temperature_ptr)`) and the Helion kernel (`temperature: torch.Tensor`) accept 0-d tensors directly.
- **Always save logs to the output folder.** When running servers, benchmarks, or evals, pipe stdout/stderr to a log file in the results directory so logs are always accessible after the run. Never discard or hide process output.
- **Pandas style**: Always use pandas (or equivalent DataFrame library) for data analysis. Never write nested loops with manual data joins when a pandas-based solution exists. Use `.query()` for row filtering, never boolean indexing (`df[df["col"] == val]`). Use `.merge()` for joins. Use `.groupby().agg()` instead of manual loops over unique values. Use `.pivot()` / `.melt()` for reshaping. Use `pd.concat()` to build DataFrames, not list-of-dicts loops.

## Writing style (README, blog post, docs)

- Single author project. Never use "we". Prefer "I" + active voice, but use passive voice when it sounds more natural.
- One sentence per line in prose sections, to make git diffs cleaner.
- Don't write `torch.compile`-d or `torch.compiled` — say "torch compiled".
- Avoid jargon like "unfused" or "lean" when simpler words work ("baseline", "Gumbel-max kernel").
- When stating speedup ranges, verify them against the actual table data. Use "generally outperforms" rather than "always" when there are exceptions.
- Never use em dashes (—). Use periods, commas, or parentheses instead.

## Blog post

The blog post lives at `~/code/tomasruizt.github.io/tomas-blog/posts/07_fused-mm-sample/index.qmd` (Quarto format).
It should be kept in sync with the README benchmark numbers.
The blog uses both "large" (V=128,256, d=8,192) and "small" (V=151,936, d=4,096) configs, presented as the outermost tabset in the kernel benchmarks section.

### Quarto conventions

- **Panel tabsets**: `::: {.panel-tabset group="name"}` with `# Tab Name` headers. The `group=` attribute synchronizes tab selection across multiple tabsets with the same group name.
- **Nested tabsets**: Use `::::` (4 colons) for the outer tabset and `:::` (3 colons) for the inner tabset. Outer tabs use `#` headers, inner tabs use `##` headers. Example:

  ```markdown
  :::: {.panel-tabset group="baseline"}
  # vs PyTorch Compiled
  ::: {.panel-tabset group="gpu"}
  ## B300

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FlashSampling/FlashSampling](https://github.com/FlashSampling/FlashSampling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
