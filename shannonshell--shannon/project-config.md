---
trigger: always_on
description: A poly-shell built on nushell, with seamless bash compatibility via a
---

# Shannon

A poly-shell built on nushell, with seamless bash compatibility via a
persistent bash subprocess. Shift+Tab cycles between nu and bash modes.
Named after Claude Shannon.

## Build

```sh
cargo build
cargo run
```

The Rust crate is at the repo root.

## Architecture

Shannon IS nushell — it copies the nushell binary source code and adds mode
dispatch for bash (via a persistent bash subprocess). Nushell's REPL handles
terminal ownership, process groups, job control, signal handling, multiline
editing, completions, and all interactive features. Shannon adds a
`ModeDispatcher` trait (defined in nu-cli) that intercepts commands when the
mode is not "nu".

### Repo structure

```
shannon/              (repo root = shannonshell crate)
├── Cargo.toml        (binary + library)
├── src/              (shannon source)
├── nushell/          (nushell source, merged into repo)
├── reedline/         (reedline source, merged into repo)
├── vendor/           (reference source code, gitignored)
├── issues/           (issue tracking with experiments)
└── scripts/          (build, install, release, sync-upstream)
```

### Merged dependencies

Nushell and reedline source is merged directly into the repo with full git
history preserved (via `git subtree add`). Shannon uses path deps to reference
their crates. No crates.io publishing needed for dependencies.

- **nushell/** — fork of `nushell/nushell`. Changes: `ModeDispatcher` trait
  in nu-cli, `BashHighlighter` in nu-cli, Shift+Tab keybinding.
- **reedline/** — fork of `nushell/reedline`. No code changes.

Upstream sync: `git subtree pull --prefix nushell upstream-nushell main`

### Source files (under `src/`)

**Copied from nushell binary (the shell):**
- `main.rs` — entry point, startup sequence (from nu binary, modified)
- `run.rs` — `run_repl()` wrapper, creates ModeDispatcher, shows banner
- `command.rs` — CLI argument parsing
- `command_context.rs` — registers all nushell commands
- `config_files.rs` — loads env.nu, config.nu, login.nu
- `signals.rs` — Ctrl+C handler via ctrlc crate
- `terminal.rs` — Unix terminal/process group acquisition
- `logger.rs` — logging setup
- `ide.rs` — IDE/LSP features
- `experimental_options.rs` — nushell experimental feature flags
- `test_bins.rs` — nushell test utilities

**Shannon-specific (engines and dispatch):**
- `lib.rs` — library exports for the shannonshell crate
- `dispatcher.rs` — `ShannonDispatcher` implementing `ModeDispatcher`
- `bash_process.rs` — `BashProcess` (persistent bash subprocess with sentinel protocol)
- `shell_engine.rs` — `ShellEngine` trait
- `shell.rs` — `ShellState` (env, cwd, exit code)
- `executor.rs` — bash `export -p` output parsing

### How command execution works

**Nu mode (default):**
1. User types a command
2. Nushell's `loop_iteration()` reads input via reedline
3. `$env.SHANNON_MODE` is "nu" — falls through to nushell's parser/evaluator
4. Nushell handles everything: parsing, execution, output, env updates

**Bash mode (bash):**
1. User types a command
2. Nushell's `loop_iteration()` reads input via reedline
3. `$env.SHANNON_MODE` is "bash" — calls `ModeDispatcher::execute()`
4. Env vars converted to strings via `env_to_strings()`
5. `BashProcess` injects env vars and cwd, writes command + sentinel to bash stdin
6. Command output streams to stdout; sentinel block parsed for env, cwd, exit code
7. Result written back to nushell's Stack; REPL continues with updated state

### Environment propagation

When switching modes, all exported environment variables and cwd are preserved:
- **Nu → Bash:** `env_to_strings()` converts nushell typed values to strings
  (using `ENV_CONVERSIONS` `to_string` closures for PATH etc.). Injected into
  bash via `export` commands.
- **Bash → Nu:** `export -p` captures env vars after each command. String env
  vars written back to Stack via `add_env_var()`. Nushell's REPL automatically
  applies `from_string` conversions on the next iteration.

### Testing

Every new feature must include tests. No feature ships without test coverage.

- **Unit tests** go in each module as `#[cfg(test)] mod tests { ... }`.
- **Integration tests** go in `tests/`.
- Use `tempfile::TempDir` for tests that need filesystem fixtures.
- `cargo test` must pass before a feature is considered done.

### Key design decisions

- **Shannon IS nushell** — the binary copies nushell's startup code (~4,600
  lines) and adds mode dispatch. Shannon gets all nushell features for free:
  job control, plugins, multiline editing, completions, hooks, etc.
- **Trait injection** — `ModeDispatcher` trait defined in nu-cli, implemented
  by `ShannonDispatcher` in shannon. Nushell's fork has a ~30-line hook in
  `loop_iteration()` that dispatches to bash when `$env.SHANNON_MODE` is
  "bash". Shannon stays the primary binary; nushell stays a dependency.
- **Strings at the boundary** — env vars cross between shells as strings.
  `env_to_strings()` and `from_string` conversions handle typed values
  (PATH as list, etc.).
- **Monorepo** — nushell and reedline source merged directly into the repo
  with full git history. Path deps, no crates.io publishing needed.
  Upstream sync via `git subtree pull` (full history preserved).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shannonshell/shannon](https://github.com/shannonshell/shannon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
