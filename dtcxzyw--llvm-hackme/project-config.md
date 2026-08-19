---
trigger: always_on
description: `llvm-hackme` is an automated LLVM correctness checking service.  See
---

# AGENTS

## Project Overview

`llvm-hackme` is an automated LLVM correctness checking service.  See
`README.md` for the user-facing overview and `INTERNALS.md` for the
architecture, state machine, and internal design.
Agents must read and maintain `INTERNALS.md` when making structural
changes.

## Project Constraints

- **Single-instance only** — only one `HackmeService` process runs against a given repository. There is no distributed coordination or multi-instance support. Do not design features that require cross-process locking, consensus, or distributed state.
- **Single comment per PR** — `_pr_tasks` (asyncio task dict) ensures at most one active processing task per PR number. `report_result()` queries both the local DB and the GitHub API before creating a new comment, and recovers by updating an existing comment if found. No DB-based comment-creation lock is needed.
- **Fuzz parallelism by multiprocessing** — The fuzzer spawns `os.cpu_count()` worker processes via `ProcessPoolExecutor`. Each worker independently runs one mutation iteration (mutate → opt → alive2). Workers run in parallel and the fuzzer stops when any worker finds a bug or the time budget expires. The sequential asyncio `max_fuzz_parallelism` config has been removed.
- **Build lock covers fuzz + hack** — `_build_lock` is intentionally held across `prepare_pr_worktree`, `build_pr_opt`, fuzzing, and the hack agent. This prevents baseline updates (which rebuild the toolchain) from running concurrently with any PR processing that uses the toolchain. Do not shrink the lock scope to release it before fuzz/hack.
- **Memory limit propagation** — `opt_memory_limit_bytes` must be passed from config through verification (`verify_reproducer` → `check_crash` / `check_miscompilation`) so bugs that only reproduce under memory pressure are not silently discarded as false negatives.
- **Build type** — LLVM must be built with `RelWithDebInfo` so crash stacktraces are meaningful. `llvm-symbolizer` must be on PATH at startup.
- **`instcombine` normalisation** — bare `instcombine` arguments (legacy PM syntax) are automatically normalised to `instcombine<no-verify-fixpoint>` everywhere opt args are parsed.

## Development Environment

- **Python dependency management** — use `uv` (see `uv.lock`). Run `uv sync` to install all dependencies (including dev). Run `uv pip install -e .` to install the package itself in editable mode. The virtualenv lives at `.venv/`.
- **Testing** — `uv run pytest tests/` or `.venv/bin/python -m pytest tests/`. Test config is in `pyproject.toml` under `[tool.pytest.ini_options]`.
- **Linting** — `uv run ruff check .`

## Repository Language Rules

- Write all repository content in English and reply to the user in the user's language.

## User Interaction Rules

- If requirements or plans are ambiguous, eliminate disagreement instead of guessing: first explore the codebase when that can answer the question, otherwise ask clarifying questions aggressively, preferably one at a time.
- Clarifying questions must drive toward shared understanding by walking the design tree and resolving decision dependencies; each question must cover purpose, constraints, success criteria, or scope boundaries as appropriate, include all viable options, avoid requiring free-form input, and state the recommended answer.
- After the user approves execution, keep going until the full planned task is complete. Do not stop for intermediate progress reports, return control while the approved goal is only partially implemented, or ask whether to continue with obvious next steps, natural follow-ups, or clear previews, summaries, or refinements unless a genuine blocker, real ambiguity, or material risk requires user input.
- If something should obviously be done now, do it instead of deferring it to "later", "next", or a "follow-up". When reviewing or refining in-progress work, immediately implement any concrete fix you understand and that is not blocked.
- Resolve encountered difficulties autonomously whenever possible.
- When handing control back to the user after task completion, end the response with `Done.`

## Design and Planning Rules

- Before proposing a design or implementation, inspect the current project context, including relevant files, documentation, and recent changes.
- State assumptions explicitly. If materially different interpretations remain after exploration, surface and resolve them instead of silently choosing one.
- For feature work, behavior changes, and other creative or architectural tasks, complete a design step before implementation. If the request is too broad for one coherent spec or plan, decompose it into smaller subprojects and handle them one at a time.
- After gathering enough context, present 2-3 viable approaches with trade-offs and a recommended option, size the design sections to the topic, and validate the design with the user before implementation.
- Before substantial implementation, define concrete success criteria and explicit checks that prove completion; prefer tests, builds, or other verifiable validation over vague goals.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dtcxzyw/llvm-hackme](https://github.com/dtcxzyw/llvm-hackme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
