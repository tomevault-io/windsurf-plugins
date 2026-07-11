---
trigger: always_on
description: **Always check [`docs/design-tenets.md`](docs/design-tenets.md) before proposing or implementing changes.** Validate that your approach aligns with the tenets — especially "don't duplicate unix," "just workspace management," and "structured output is the contract."
---

# wsp - Multi-Repo Workspace Manager

**Always check [`docs/design-tenets.md`](docs/design-tenets.md) before proposing or implementing changes.** Validate that your approach aligns with the tenets — especially "don't duplicate unix," "just workspace management," and "structured output is the contract."

## Quick Reference

| What | Where |
|------|-------|
| All CLI commands | [`skills/wsp-manage/SKILL.md`](skills/wsp-manage/SKILL.md) (auto-generated) |
| Architecture & module map | [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) |
| Removal safety algorithm | [`docs/features/removal-safety.md`](docs/features/removal-safety.md) |
| Release workflow | `/wsp-release` skill |

## After Cloning

```bash
just setup    # install pre-commit hook (run once; prevents fmt/lint failures in CI)
```

## Build & Test

```bash
just          # check (fmt + clippy)
just build    # release binary (runs check, regenerates SKILL.md)
just test     # all tests
just ci       # full pipeline (check + build + test + SKILL.md freshness)
just fix      # auto-fix fmt and lint
```

Run `just fix` before `just ci` when you've made changes — `just ci` starts with `cargo fmt --check` and fails immediately on unformatted code. Running `just ci` before pushing is an optimization; CI will catch failures on the PR automatically, but it saves a round-trip.

The `codegen` feature gates `wsp generate`, which introspects clap to produce SKILL.md. `just check` runs clippy with and without it. Adding a command or output struct updates SKILL.md automatically on next `just build`.

## Data Storage

- Config: `~/.local/share/wsp/config.yaml`
- Mirrors: `~/.local/share/wsp/mirrors/<host>/<user>/<repo>.git/`
- Workspaces: `~/dev/workspaces/<name>/` with `.wsp.yaml` metadata
- GC (deferred deletions): `~/.local/share/wsp/gc/<name>__<timestamp>/` with `.wsp-gc.yaml`

## File Locking

Use `filelock::with_config()` / `with_metadata()` / `with_template()` for all read-modify-write operations. Never call `load` → modify → `save` directly outside of tests. Keep locks short: do not hold during network I/O. Use the 3-phase pattern: snapshot under lock → slow I/O → update under lock with re-check.

## Security

- **Shell completions** (`completion.rs`): escape user values for target shell. POSIX: `'` → `'\''`. Fish: `'` → `\'`. Completion must never fail at shell startup — degrade gracefully.
- **Path traversal**: new code building paths from user input must use `giturl::validate_component()`.
- **Git config keys**: validate with `config::validate_git_config_key()` before writing. One canonical denylist in `config.rs` — do not add a second one.
- `#![deny(unsafe_code)]` enforced at both crate roots.
- **Platform code**: guard with `#[cfg(unix)]` / `#[cfg(windows)]`. See `agentmd.rs` for the pattern.

## Naming

Product: binary `wsp`, metadata `.wsp.yaml`, env `WSP_SHELL`, shell vars `wsp_bin`/`wsp_root`/`wsp_dir`, data dir `~/.local/share/wsp/`. Internal Rust names (`ws_dir`, `ws_bin`) are shorthand, not product identifiers.

**CLAUDE.md is a symlink to AGENTS.md** — do not replace the symlink with a regular file.

**Before proposing a new command name**, check the open GitHub issues (labels P1--P4) — planned command names are reserved there. A name collision means either the existing issue needs to be closed/updated first, or a different name is needed.

## Conventions

- Git ops via `std::process::Command`, not libgit2
- Table-driven tests; property-based tests where applicable
- YAML config with `serde_yaml_ng`; error handling with `anyhow`
- Git output with tty formatting: pass `--color=always` gated on `stdout().is_terminal() && !is_json`
- Read-only commands get `[read-only]` in `.about()`. Every flag accepting known values needs an `ArgValueCandidates` completer.
- Clap dispatch: only match primary command name (e.g., `Some(("ls", m))` — not aliases).
- **Workspace positional must be optional**: every command that operates on a single workspace takes `<workspace>` as an optional positional with CWD detection fallback via `workspace::detect()`. When a command also has a second positional (e.g. `describe [workspace] <text>`), keep both as named args, make the second optional in clap, and in `run()` treat the single-arg case as "detect workspace from CWD, first arg is the payload." Use `.override_usage()` to show the correct semantic order. See `describe.rs` and `rename.rs` for the pattern. A `test_workspace_arg_is_optional` test in `cli/mod.rs` enforces this.
- **Boolean "mode" flags + positional args**: `ArgGroup` only enforces mutual exclusion among named flags — it does not cover positional args. If a mode flag (e.g. `--empty`) is incompatible with positionals (e.g. repo args), add an explicit early bail in `run()` or use `.conflicts_with("repos")` on the flag's `Arg` definition.
- **`--force` vs `--yes` — two distinct flags, two distinct jobs**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jganoff/wsp](https://github.com/jganoff/wsp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
