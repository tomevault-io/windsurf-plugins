---
trigger: always_on
description: This document explains the intent of Asimeow, the key behaviours exposed via its CLI, and the internal "agents" (components) that collaborate to perform automatic Time Machine exclusions on macOS.
---

## Asimeow agents: goal, behaviours, and how it works

This document explains the intent of Asimeow, the key behaviours exposed via its CLI, and the internal "agents" (components) that collaborate to perform automatic Time Machine exclusions on macOS.

### Goal

- **Primary goal**: Identify development artifacts (e.g., `node_modules`, `target`, `dist`) inside project folders and exclude them from macOS Time Machine backups to save space and speed up backups.
- **Approach**: Recursively traverse configured root folders, match project indicators (like `package.json`, `Cargo.toml`), and exclude rule-defined directories using `tmutil`.

## Behaviours (what the tool does)

- **Automatic scan and exclude**: Walks directories under configured `roots`, matches `rules` via `file_match` glob patterns, and excludes each rule's `exclusions` directories when found.
- **Ignore directories**: Skips directories matching `ignore` patterns (glob on the directory name, e.g., `.git`).
- **Manual control**:
  - `list [path]`: Show Time Machine exclusion status for a directory or a single path.
  - `exclude <path>`: Force-exclude a directory or file via `tmutil addexclusion`.
  - `include <path>`: Remove exclusion via `tmutil removeexclusion`.
- **Initialization**: `init` generates a default `config.yaml` either locally or at `~/.config/asimeow/config.yaml` with common presets for many ecosystems.
- **Reporting**: Prints concise status lines (✅ newly excluded, 🟡 already excluded) and summary totals (processed paths, exclusions found, newly excluded) when relevant or in verbose mode.

## How it works (internal architecture)

### CLI and entrypoint

- File: `src/main.rs`
- **Argument parsing**: Uses `clap` for flags and subcommands:
  - `-c, --config` path (defaults to auto-detection if left as the default value `config.yaml`)
  - `-v, --verbose`
  - `-t, --threads <N>` worker threads for traversal
  - Subcommands: `init`, `version`, `list`, `exclude`, `include`
- **Execution flow**:
  1. If a subcommand is provided, execute it immediately and exit.
  2. Otherwise, load configuration (auto-detect path) and run the explorer with the selected thread count.

### Configuration agent

- File: `src/config.rs`
- **Schema**:
  - `Config { roots: Vec<Root>, ignore: Vec<String>, rules: Vec<Rule> }`
  - `Root { path: String }`
  - `Rule { name: String, file_match: String, exclusions: Vec<String> }`
- **Responsibilities**:
  - `create_default_config(local, path)`: creates a YAML config with common rules (Node, Rust, Python, etc.). Ensures directories and writes file.
  - `find_config_file(specified)`: resolves config from explicit path, `./config.yaml`, or `~/.config/asimeow/config.yaml`.
  - `load_config(path, verbose)`: reads, parses YAML, prints loaded rules in verbose mode, and validates `roots`.
  - `expand_tilde(path)`: resolves `~/` to the user home directory.

### Explorer agent (orchestrator)

- File: `src/explorer.rs`
- **State model**:
  - `State` with thread-safe counters and a shared queue: `folder_queue`, `exclusion_found`, `processed_paths`, `active_tasks`, `processing_complete`, `newly_excluded` (all protected by `RwLock`).
  - Simple in-memory work queue (`Vec<PathBuf>`) managed under locks; workers pop from the front.
- **Workers**:
  - `run_workers(state, rules, thread_count, verbose, ignore_patterns)`: spawns `thread_count` threads. Each thread repeatedly pulls a path from the queue and calls `process_path()` until the queue empties and no tasks are active.
  - Completion is detected when the queue is empty and `active_tasks == 0`, then `processing_complete` is set.
- **Traversal**:
  - `run_explorer(config, threads, verbose)`: enqueues each `root` (with `~` expansion), then starts workers. After completion, prints totals.
  - `process_path(path, state, rules, verbose, ignore_patterns)`:
    - Validates path exists and is a directory.
    - Applies ignore checks against the current directory name using glob patterns from `ignore`.
    - Reads entries once, collecting subdirectories while evaluating rule matches.
    - For each file/entry in the current directory, matches `rule.file_match` using glob semantics (case-insensitive via lowercase comparisons).
    - On a rule match:
      - For each `exclusions` item, builds `exclusion_path = current_dir / exclusion` and attempts exclusion.
      - Records `directory_to_ignore` to avoid descending into newly excluded directories.
      - Special-case: if `exclusions` contains "." or "..", the function returns early (treat as do-not-descend behaviour for current or parent).
    - Finally, enqueues subdirectories that are not ignored and not among the excluded dir names.

### Time Machine integration agent

- File: `src/explorer.rs`
- **Functions**:
  - `is_excluded_from_timemachine(path)`: runs `tmutil isexcluded <path>` and parses `[Excluded]`.
  - `exclude_from_timemachine(path)`: runs `tmutil addexclusion <path>`; returns false if already excluded.
  - `include_in_timemachine(path)`: runs `tmutil removeexclusion <path>`; returns false if already included.
- **User-facing commands**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mdnmdn/asimeow](https://github.com/mdnmdn/asimeow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
