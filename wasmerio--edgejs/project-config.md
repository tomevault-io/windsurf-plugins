---
trigger: always_on
description: This repo contains an N-API implementation backed by QuickJS. Most of the active
---

# EdgeJS N-API QuickJS Notes

This repo contains an N-API implementation backed by QuickJS. Most of the active
work has been in the `napi` submodule/worktree, especially:

- `napi/quickjs/src/js_native_api_quickjs.cc`
- `napi/quickjs/src/unofficial_napi.cc`
- `napi/quickjs/src/internal/`
- `napi/quickjs/tests/`
- `napi/tests/js-native-api/`

## QuickJS WASIX Resume Context

When resuming this work, start with:

```text
plans/quickjs-wasm/development/README.md
```

That file indexes the development phases:

- `001_merge_analysis.md`: comparison with the other QuickJS branch and integration plan.
- `002_native_bootstrap_contextify.md`: native Edge QuickJS bootstrap and `ContextifyScript` fix.
- `003_repl_tty_readline.md`: REPL TTY/readline troubleshooting.
- `004_promise_hooks_microtasks.md`: QuickJS promise hooks and microtask/job draining.
- `005_wasix_wasmer_http.md`: WASIX/Wasmer bootstrap, Atomics, and HTTP stream listener fix.
- `006_framework_app_adapters.md`: Astro, Vite, and Next.js app adapter notes.
- `007_framework_standalone_builds.md`: framework standalone build notes and remaining runtime
  blockers.
- `008_runtime_change_containment_rollback.md`: shared runtime rollback containment, native
  compatibility relocation, and QuickJS WASIX build/linkage notes.

Current useful state:

- Native QuickJS-backed Edge CLI can bootstrap and run the HTTP echo server.
- REPL input works with persistent history after the promise hook/microtask fix.
- WASIX Edge QuickJS can run under Wasmer and handle HTTP requests with `--net`.
- `quickjs-wasm/build.sh` currently builds `build-quickjs-wasix/edge.wasm` and
  `edgejs.wasm`, and its final no-N-API-imports check passes.
- The root `wasmer.toml` publishes/uses `sadhbh-c0d3/edgejs-quickjs` at
  `0.0.1`, module `edge`, source `build-quickjs-wasix/edgejs.wasm`.
- Framework app notes use anonymized paths: `~/src/astro-app`,
  `~/src/vite-app`, and `~/src/next-app`.

## Plans Documentation Workflow

Before starting a new task, always list the plan tree recursively and look for
existing information:

```sh
find /Users/sadhbh/src/dev/edgejs/plans -type f -print
rg -n "<relevant terms>" /Users/sadhbh/src/dev/edgejs/plans
```

Read the relevant existing plan, development note, or troubleshooting note
before changing code. While working, keep existing information current: if the
task discovers new facts about an existing topic, update the existing note
instead of creating a duplicate.

If the context window reaches 90% while work is in progress, create a new
development task note under:

```text
plans/quickjs-wasm/development/NNN_<meaningful_name>.md
```

Include all information needed to continue the current task: user requests,
review comments being addressed, files changed, verification already run,
known failures, and the next concrete steps.

### Generate PDF Documentation

When asked to "Generate PDF documentation", build a polished white-paper/book
PDF from `plans/quickjs-wasm` and all of its subdirectories.

Use this process:

1. List the plan tree recursively and search the plans for relevant context
   before generating the document.
2. Generate temporary Markdown and LaTeX under `/private/tmp`, leaving the
   source plan notes untouched.
3. Organize the book by knowledge structure, not raw file order: program
   definition, chronological development narrative, cleanup/containment
   subtasks, troubleshooting registry, Astro SSR, Vite app, Next.js, and Wasmer
   deploy/WASIX packaging.
4. Preserve all source-note information as chapters or chapter sections, and
   include source paths for traceability.
5. Use the title `EdgeJS QuickJS WASIX`, author
   `Sonia Sadhbh Kolasinska in collaboration with Christoph Herzog, Wasmer`,
   the current date, and an abstract.
6. Render through temporary LaTeX with Pandoc and XeLaTeX, rerunning XeLaTeX as
   needed for the table of contents.
7. Preserve literal tilde characters in paths and code examples; do not rewrite
   `~` as math such as `$\sim$`.
8. Write the final PDF into `plans/quickjs-wasm/`.

## Experimental Rules

### Experimental 001: Parallel Development Subtasks

For larger development work, split the task into a development task directory:

```text
plans/quickjs-wasm/development/dev_<number>_<meaningful-name>/<subtask-number>_<meaningful-name>.md
```

Each subtask note should record scope, dependencies, write ownership, status,
verification expectations, and enough context for an independent worker to
continue safely. Spawn workers intelligently based on dependency order: only run
parallel workers for subtasks with disjoint write sets or read-only checks, and
make each worker aware that others may be active in the same codebase.

Use this heuristic when deciding where documentation belongs:

- Development task: broad implementation progress, integration work, runtime
  design, refactors, or milestone notes. Write or update a numbered development
  note under:

```text
plans/quickjs-wasm/development/NNN_<meaningful_name>.md
```

- Troubleshooting issue: an observed failure, crash, regression, compatibility
  gap, or focused diagnostic trail. Write or update a numbered issue note under
  the app-specific troubleshooting directory:

```text

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wasmerio/edgejs](https://github.com/wasmerio/edgejs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
