---
trigger: always_on
description: We only care about the ACP backend and the code that compiles into the nori bin. We do not care about the default codex backend or the code that compiles into the codex bin. Make sure all changes and responses are aligned to this critical fact. For example, if I ask 'add notifications to the cli', you should assume I mean 'add notification support to the ACP backend for the nori cli'.
---

# We only care about the ACP backend

We only care about the ACP backend and the code that compiles into the nori bin. We do not care about the default codex backend or the code that compiles into the codex bin. Make sure all changes and responses are aligned to this critical fact. For example, if I ask 'add notifications to the cli', you should assume I mean 'add notification support to the ACP backend for the nori cli'.

# Rust/nori-rs

In the nori-rs folder where the rust code lives:

- Crate names are prefixed with `codex-`. For example, the `core` folder's crate is named `codex-core`
- When using format! and you can inline variables into {}, always do that.
- Install any commands the repo relies on (for example `just`, `rg`, or `cargo-insta`) if they aren't already available before running instructions here.
- Never add or modify any code related to `CODEX_SANDBOX_NETWORK_DISABLED_ENV_VAR` or `CODEX_SANDBOX_ENV_VAR`.
  - You operate in a sandbox where `CODEX_SANDBOX_NETWORK_DISABLED=1` will be set whenever you use the `shell` tool. Any existing code that uses `CODEX_SANDBOX_NETWORK_DISABLED_ENV_VAR` was authored with this fact in mind. It is often used to early exit out of tests that the author knew you would not be able to run given your sandbox limitations.
  - Similarly, when you spawn a process using Seatbelt (`/usr/bin/sandbox-exec`), `CODEX_SANDBOX=seatbelt` will be set on the child process. Integration tests that want to run Seatbelt themselves cannot be run under Seatbelt, so checks for `CODEX_SANDBOX=seatbelt` are also often used to early exit out of tests, as appropriate.
- Always collapse if statements per https://rust-lang.github.io/rust-clippy/master/index.html#collapsible_if
- Always inline format! args when possible per https://rust-lang.github.io/rust-clippy/master/index.html#uninlined_format_args
- Use method references over closures when possible per https://rust-lang.github.io/rust-clippy/master/index.html#redundant_closure_for_method_calls
- Do not use unsigned integer even if the number cannot be negative.
- When possible, make `match` statements exhaustive and avoid wildcard arms.
- Do not create small helper methods that are referenced only once.
- When making a change that adds or changes an API, ensure that the documentation in the `docs/` folder is up to date if applicable.
- Avoid large modules:
  - Prefer adding new modules instead of growing existing ones.
  - Target Rust modules under 500 LoC, excluding tests.
  - If a file exceeds roughly 800 LoC, add new functionality in a new module instead of extending the existing file unless there is a strong documented reason not to.
  - This rule applies especially to high-touch files such as `nori-rs/tui/src/app.rs`, `nori-rs/tui/src/bottom_pane/chat_composer.rs`, `nori-rs/tui/src/bottom_pane/footer.rs`, `nori-rs/tui/src/chatwidget.rs`, `nori-rs/tui/src/bottom_pane/mod.rs`, and similarly central orchestration modules.
  - When extracting code from a large module, move the related tests and module/type docs toward the new implementation so the invariants stay close to the code that owns them.
- When running Rust commands (e.g. `just fix` or `cargo test`) be patient and never try to kill them using the PID. Rust lock contention can make execution slow; this is expected.

Run `just fmt` (in `nori-rs` directory) automatically after you have finished making Rust code changes; do not ask for approval to run it. Additionally, run the tests:

1. Run the test for the specific project that was changed. For example, if changes were made in `nori-rs/tui`, run `cargo test -p nori-tui`.
2. Once those pass, if any changes were made in common, core, or protocol, run the complete test suite with `cargo test`. Avoid `--all-features` for routine local runs because it expands the build matrix and can significantly increase `target/` disk usage; use it only when you specifically need full feature coverage.
3. If any changes were made in tui, cli, or acp, run `cargo build --bin nori && cargo test -p tui-pty-e2e`. The E2E tests require the `nori` binary (from the `cli` crate), not `nori-tui`.

Before finalizing a change to `nori-rs`, run `just fix -p <project>` (in `nori-rs` directory) to fix any linter issues in the code. Prefer scoping with `-p` to avoid slow workspace‑wide Clippy builds; only run `just fix` without `-p` if you changed shared crates. Do not re-run tests after running `fix` or `fmt`.

## TUI style conventions

See `nori-rs/tui/styles.md`.

## TUI code conventions

### Styling (ratatui)

- Prefer Stylize helpers (`"text".dim()`, `.bold()`, `.cyan()`, `.underlined()`) over manual `Style`. Chain for readability (e.g., `url.cyan().underlined()`).
- Basic spans: `"text".into()`. Styled: `"text".red()`. Use `Line::from(…)` / `Span::from(…)` only when target type is ambiguous.
- `vec![…].into()` for lines when type is obvious; `Line::from(vec![…])` otherwise.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tilework-tech/nori-cli](https://github.com/tilework-tech/nori-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
