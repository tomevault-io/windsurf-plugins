---
trigger: always_on
description: In the codex-rs folder where the rust code lives:
---

# Rust/codex-rs

In the codex-rs folder where the rust code lives:

- Crate names are prefixed with `codex-`. For example, the `core` folder's crate is named `codex-core`
- When using format! and you can inline variables into {}, always do that.
- Install any commands the repo relies on (for example `just`, `rg`, or `cargo-insta`) if they aren't already available before running instructions here.
- For release CLI builds, always use `make release-codex` or `just release-codex` from repo root instead of raw `cargo build --release`. These targets clear `CARGO_PROFILE_RELEASE_LTO`, `CARGO_PROFILE_RELEASE_CODEGEN_UNITS`, `CARGO_PROFILE_RELEASE_DEBUG`, and `CARGO_PROFILE_RELEASE_STRIP` before build, then install to `/Users/chenwenjie/bin/codex` by default.
- Never add or modify any code related to `CODEX_SANDBOX_NETWORK_DISABLED_ENV_VAR` or `CODEX_SANDBOX_ENV_VAR`.
  - You operate in a sandbox where `CODEX_SANDBOX_NETWORK_DISABLED=1` will be set whenever you use the `shell` tool. Any existing code that uses `CODEX_SANDBOX_NETWORK_DISABLED_ENV_VAR` was authored with this fact in mind. It is often used to early exit out of tests that the author knew you would not be able to run given your sandbox limitations.
  - Similarly, when you spawn a process using Seatbelt (`/usr/bin/sandbox-exec`), `CODEX_SANDBOX=seatbelt` will be set on the child process. Integration tests that want to run Seatbelt themselves cannot be run under Seatbelt, so checks for `CODEX_SANDBOX=seatbelt` are also often used to early exit out of tests, as appropriate.
- Always collapse if statements per https://rust-lang.github.io/rust-clippy/master/index.html#collapsible_if
- Always inline format! args when possible per https://rust-lang.github.io/rust-clippy/master/index.html#uninlined_format_args
- Use method references over closures when possible per https://rust-lang.github.io/rust-clippy/master/index.html#redundant_closure_for_method_calls
- When possible, make `match` statements exhaustive and avoid wildcard arms.
- When writing tests, prefer comparing the equality of entire objects over fields one by one.
- When making a change that adds or changes an API, ensure that the documentation in the `docs/` folder is up to date if applicable.
- If you change `ConfigToml` or nested config types, run `just write-config-schema` to update `codex-rs/core/config.schema.json`.
- If you change Rust dependencies (`Cargo.toml` or `Cargo.lock`), run `just bazel-lock-update` from the
  repo root to refresh `MODULE.bazel.lock`, and include that lockfile update in the same change.
- After dependency changes, run `just bazel-lock-check` from the repo root so lockfile drift is caught
  locally before CI.
- Do not create small helper methods that are referenced only once.

Run `just fmt` (in `codex-rs` directory) automatically after you have finished making Rust code changes; do not ask for approval to run it. Additionally, run the tests:

1. Run the test for the specific project that was changed. For example, if changes were made in `codex-rs/tui`, run `cargo test -p codex-tui`.
2. Once those pass, if any changes were made in common, core, or protocol, run the complete test suite with `cargo test` (or `just test` if `cargo-nextest` is installed). Avoid `--all-features` for routine local runs because it expands the build matrix and can significantly increase `target/` disk usage; use it only when you specifically need full feature coverage. project-specific or individual tests can be run without asking the user, but do ask the user before running the complete test suite.

Before finalizing a large change to `codex-rs`, run `just fix -p <project>` (in `codex-rs` directory) to fix any linter issues in the code. Prefer scoping with `-p` to avoid slow workspace‑wide Clippy builds; only run `just fix` without `-p` if you changed shared crates. Do not re-run tests after running `fix` or `fmt`.

## TUI style conventions

See `codex-rs/tui/styles.md`.

## TUI code conventions

- Use concise styling helpers from ratatui’s Stylize trait.
  - Basic spans: use "text".into()
  - Styled spans: use "text".red(), "text".green(), "text".magenta(), "text".dim(), etc.
  - Prefer these over constructing styles with `Span::styled` and `Style` directly.
  - Example: patch summary file lines
    - Desired: vec!["  └ ".into(), "M".red(), " ".dim(), "tui/src/app.rs".dim()]

### TUI Styling (ratatui)

- Prefer Stylize helpers: use "text".dim(), .bold(), .cyan(), .italic(), .underlined() instead of manual Style where possible.
- Prefer simple conversions: use "text".into() for spans and vec![…].into() for lines; when inference is ambiguous (e.g., Paragraph::new/Cell::from), use Line::from(spans) or Span::from(text).
- Computed styles: if the Style is computed at runtime, using `Span::styled` is OK (`Span::from(text).set_style(style)` is also acceptable).
- Avoid hardcoded white: do not use `.white()`; prefer the default foreground (no color).
- Chaining: combine helpers by chaining for readability (e.g., url.cyan().underlined()).
- Single items: prefer "text".into(); use Line::from(text) or Span::from(text) only when the target type isn’t obvious from context, or when using .into() would require extra type annotations.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stellarlinkco/codex](https://github.com/stellarlinkco/codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
