---
trigger: always_on
description: Guidance for coding agents working in this repository.
---

# CLAUDE.md

Guidance for coding agents working in this repository.

## Repository purpose

`tandem` is a git-aware ticket coordination system for AI agents in a monorepo.

It is designed to work across branches and git worktrees. Ticket state is stored in the repository and exposed via a deterministic `tndm` CLI. Repo-local ticket files are the system of record; no central service is required.

Start with:
- Product vision: `docs/vision.md`
- Design decisions: `docs/decisions.md`
- Architecture overview: `docs/architecture.md`

## Project structure

- `crates/tandem-core` — domain logic + validation + core ports; must remain IO-free.
- `crates/tandem-storage` — filesystem storage adapter implementing core ports.
- `crates/tandem-repo` — git/worktree awareness adapter implementing core ports.
- `crates/tandem-cli` — CLI crate producing `tndm`; the only crate allowed to depend on `clap`.
- `crates/xtask` — dev tooling, including `cargo xtask check-arch`.
- `docs/` — product and architecture docs; start with `docs/vision.md`, `docs/decisions.md`, and `docs/architecture.md`.
- `target/` — local build output; do not commit.
- `plugins/supi-flow` — PI-only extension; see `plugins/supi-flow/CLAUDE.md` for detailed guidance.

## Workspace invariants (Rust)

Enforced dependency direction (validated by `cargo xtask check-arch`; see `crates/xtask/src/main.rs`):
- `tandem-core` has no workspace-crate dependencies.
- `tandem-storage -> tandem-core`
- `tandem-repo -> tandem-core`
- `tandem-cli -> tandem-core + tandem-storage + tandem-repo`
- Only `tandem-cli` may depend on `clap`.

Sources of truth (enforced by tooling):
- Architecture boundaries and “clap only in CLI”: `crates/xtask/src/main.rs`
  (invoked via `cargo xtask check-arch`; alias in `.cargo/config.toml`)
- `tandem-core` IO bans: `clippy.toml`
- No `unsafe`: workspace lints in root `Cargo.toml`

If you add or rename workspace crates, update `crates/xtask/src/main.rs` to keep the workspace crate list and edge rules current.

Cross-crate shared constants and defaults go in `tandem-core` — it is the dependency root, so placing values like `DEFAULT_CONTENT_TEMPLATE` there avoids duplicating string literals across crates.

Product vision lives in `docs/vision.md`; design decisions in `docs/decisions.md`. Avoid encoding future plans here.

## Common development commands

Tooling is managed via `mise` (except Rust itself — Rust is pinned by `rust-toolchain.toml` and managed via `rustup`, not mise). Install via Homebrew for production: `brew install mrclrchtr/tap/tandem-cli`.

```sh
./tndm-dev fmt --check  # verify canonical .tndm formatting after serializer/CLI format changes

mise run check     # fmt + compile + arch + clippy + test (all-in-one)
mise run fix       # auto-fix formatting + clippy suggestions

hk run check        # same linters, fix=false
hk run fix          # same linters, fix=true (fmt + clippy --fix)
```

## supi-flow plugin

The `plugins/supi-flow/` directory contains a PI-only extension (not a Claude Code plugin) that implements a spec-driven workflow (brainstorm → plan → apply → archive) coupled to TNDM ticket coordination. See `plugins/supi-flow/CLAUDE.md` for full guidance.

The plugin tools wrap the `tndm` CLI directly — update the CLI help text when changing behavior.

## Git hooks and `hk`

- `cargo-clippy` runs in hk `pre-commit`, `pre-push`, and `check`.
- `cargo-test` is intentionally not in `hk.pkl`; use `mise run test` (runs `cargo test --workspace --locked`).
- Renovate updates `hk.pkl`. If hk-related checks fail after a version bump, update `hk` in `mise.toml` and run `mise install` to refresh `mise.lock`.

## Verification shortcuts

- After changing ticket serialization, formatting, or canonical TOML output, run `./tndm-dev fmt --check`.

## Coding and testing conventions

- `rustfmt` is the formatter; `clippy` runs with warnings treated as errors.
- `unsafe` is forbidden (`[lints.rust] unsafe_code = "forbid"`).
- Use Rust’s built-in test harness (`#[test]`).
- Prefer unit tests colocated with the code (`mod tests { ... }`); add integration tests under `tests/` when needed.
- Keep tests deterministic: no network access and stable temp paths.
- `crates/tandem-cli` normalizes bare ticket IDs in `show`, `update`, `sync`, `doc create`, and `--depends-on` using `.tndm/config.toml` `[id].prefix`; keep explicit `ticket create --id ...` behavior unchanged.
- To test configurable ticket-ID prefix behavior, write `.tndm/config.toml` in a temp repo and cover bare-ID shorthand in `crates/tandem-cli/tests/ticket_cli_tests.rs`.
- `#[serde(flatten)]` on two structs sharing a field name (e.g., `TicketMeta` + `TicketState` both flattened in `TicketJsonEntry`) causes duplicate-key errors. Use `#[serde(skip)]` or extract a shared parent field.
- `string_enum!` macro in `crates/tandem-core/src/ticket/mod.rs` — use for new string-backed enums; generates `parse()`, `as_str()`, `FromStr`, `Display`, `Serialize` from variant→str mapping (e.g., `InProgress => "in_progress"`)
- `string_enum!` variants: the variant name's `snake_case` must match its `$str` literal — `Serialize` (derive + `rename_all`) and `Display` (`as_str()`) output will diverge silently otherwise. Tests in `macro_generated_impls` catch mismatches.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrclrchtr/tandem](https://github.com/mrclrchtr/tandem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
