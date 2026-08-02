---
trigger: always_on
description: Hancho is a single-file Python build system (similar to Make/Ninja/Bazel) that uses
---

# Working with Claude on Hancho

## Project Overview
Hancho is a single-file Python build system (similar to Make/Ninja/Bazel) that uses
`.hancho` files as build scripts. The entire implementation lives in `hancho.py` (~2200
lines, no third-party dependencies, Python 3.12+).

The pitch: Ninja's speed and simplicity combined with Bazel's expressive Python-like
syntax, but lightweight and install-free. You drop `hancho.py` into your repo and run it;
there is no install step. Build scripts are plain Python, so they can run arbitrary code.
(Consequently Hancho is **not a sandbox** - a build script can do anything Python can do.)

## Interaction Guidelines

### Autonomous Work Is Confined to `sandbox/`
- The **only** filesystem changes Claude may make without explicit, specific permission are
  inside the `sandbox/` folder (gitignored). Experiments, scratch scripts, generated test
  builds, and trial `.hancho` files all go there.
- **Anywhere outside `sandbox/`** - `hancho.py`, `examples/`, `tests/`, `tutorial/`, `docs/`,
  `CLAUDE.md`, etc. - requires an explicit request before editing or creating files.
- This is a hard boundary, not a default. "Go ahead" on a sandbox task does not extend to
  touching the rest of the repo.

### Don't Be Proactive
- **Don't** automatically run commands, tests, or investigations unless explicitly asked
- **Don't** explore branches, check diffs, or analyze code without a request
- **Wait** for explicit requests before taking action
- Focus on discussion and answering questions until given a specific task

### When Running Tests
- Run 'python -m unittest' in the root of the repo.
- Always check that tests pass after changes

### Code Style
- The codebase uses pylint with some disabled checks (see top of hancho.py)
- No emojis in code or output unless explicitly requested
- Keep changes minimal and focused
- `hancho.py` is organized into `# region` / `# endregion` sections (Log, Colors, Utils,
  Path, Dict, Options, Task, Expander, Tracer, Loader, Runner, init/main).

## How Hancho Works

### The mental model
A build script creates **Tasks**. Each Task is a config (a `Dict`) describing a command to
run, its input files, and its output files. Passing one Task as another Task's input field
creates a dependency edge. Hancho assembles all Tasks into a dependency graph and runs them
in parallel via asyncio, skipping any whose outputs are already up to date.

### Build scripts (`.hancho` files)
- A `.hancho` file is just a Python module that is `exec`'d with a special module dict that
  has `hancho` and `__file__` pre-injected. The `.hancho` suffix is conventional, not
  required.
- To use Hancho you either run `hancho.py` in a directory containing `build.hancho`, or
  `import hancho` from your own Python and drive it directly.
- The root script is loaded as a **repo** (see below). It can pull in other scripts with
  `hancho.load(...)` (same repo) or `hancho.repo(...)` (new subrepo).

### Config, the `Dict` class, and merging
Almost everything in Hancho is a `Dict` - a `dict` subclass with three important behaviors:
1. **Dot access**: `config.command` is the same as `config["command"]`.
2. **Recursive merge on construction**: `Dict(a, b, c, key=val)` merges left-to-right. The
   **rightmost non-None value wins**; two Dicts at the same key are merged recursively;
   collections/mappings are deep-copied so Dicts don't alias each other.
3. `Tool` is just an alias subclass of `Dict` - identical behavior, different name for
   readability/debugging. By convention `Tool` holds a reusable command template and `Task`
   instances specialize it with concrete inputs/outputs.


# Every script context has its own config.

Ignore the below - we're back to 'config' being a preset global variable in scripts because making
"hancho.config" track the top script is a pain in the butt. Update this when you can.

# Every script context has its own config. `hancho.config` (resolved via a module-level
# `__getattr__` that reads a `contextvars.ContextVar`) returns the config for the currently
# executing script. When you call `hancho.Task(...)`, the new task's config is
# `Dict(current_script_config, *args, **kwargs)` - i.e. the script's config is automatically
# folded in. This is why fields like `gcc_flags` defined once in a script are visible to every
# Task in it.

### Fields with special prefixes (`in_` / `out_`)
Field naming drives dependency tracking:
- `in_*` fields are **input files**. If a value is a `Task` (or list containing Tasks),
  Hancho awaits that task and substitutes its output files - this is what wires the graph.
- `out_*` fields are **output files**. They are relocated under `build_dir`.
- `in_depfile` is a special case: a single compiler-generated dependency file (`.d`) that is
  treated as *both* an output (it's written under `build_dir`) and an input (its listed
  headers are checked for the rebuild decision). Supports `gcc` (`-MMD`) and `msvc`
  (`/sourceDependencies`) formats via the `depformat` field.
- Helper predicates in `Task`: `is_input_field`, `is_output_field`, `is_depfile_field`,
  `is_io_field`.

### Text expansion (the `{macro}` system)
Templates look like Python f-strings but are lazier and more powerful (see the `Expander`
region):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aappleby/hancho](https://github.com/aappleby/hancho) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
