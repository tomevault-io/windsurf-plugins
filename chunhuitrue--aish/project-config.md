---
trigger: always_on
description: This program was cloned from codex, which is a coding agent. However, it needs to be refactored into a shell command helper tool called "aish" that is completely independent from OpenAI/ChatGPT (ChatGPT models can be used, but only as one of many LLMs, not in a special relationship with OpenAI). The basic functionality of aish is: when users are working in the shell and occasionally forget or don't know how to write complex shell commands for a specific task, they can run aish, ask the AI in a d
---

# aish
This program was cloned from codex, which is a coding agent. However, it needs to be refactored into a shell command helper tool called "aish" that is completely independent from OpenAI/ChatGPT (ChatGPT models can be used, but only as one of many LLMs, not in a special relationship with OpenAI). The basic functionality of aish is: when users are working in the shell and occasionally forget or don't know how to write complex shell commands for a specific task, they can run aish, ask the AI in a dialog, receive shell commands from the AI, and then execute them.Currently, the project is still in the refactoring process, so the codebase contains a mix of original codex and aish components.

# Rust/aish-rs

In the aish-rs folder where the rust code lives:

- Crate names are prefixed with `aish-`. For example, the `core` folder's crate is named `aish-core`
- When using format! and you can inline variables into {}, always do that.
- Install any commands the repo relies on (for example `just`, `rg`, or `cargo-insta`) if they aren't already available before running instructions here.
- Never add or modify any code related to `AISH_SANDBOX_NETWORK_DISABLED_ENV_VAR` or `AISH_SANDBOX_ENV_VAR`.
  - You operate in a sandbox where `AISH_SANDBOX_NETWORK_DISABLED=1` will be set whenever you use the `shell` tool. Any existing code that uses `AISH_SANDBOX_NETWORK_DISABLED_ENV_VAR` was authored with this fact in mind. It is often used to early exit out of tests that the author knew you would not be able to run given your sandbox limitations.
  - Similarly, when you spawn a process using Seatbelt (`/usr/bin/sandbox-exec`), `AISH_SANDBOX=seatbelt` will be set on the child process. Integration tests that want to run Seatbelt themselves cannot be run under Seatbelt, so checks for `AISH_SANDBOX=seatbelt` are also often used to early exit out of tests, as appropriate.
- Always collapse if statements per https://rust-lang.github.io/rust-clippy/master/index.html#collapsible_if
- Always inline format! args when possible per https://rust-lang.github.io/rust-clippy/master/index.html#uninlined_format_args
- Use method references over closures when possible per https://rust-lang.github.io/rust-clippy/master/index.html#redundant_closure_for_method_calls
- When writing tests, prefer comparing the equality of entire objects over fields one by one.
- When making a change that adds or changes an API, ensure that the documentation in the `docs/` folder is up to date if applicable.

Run `just fmt` (in `aish-rs` directory) automatically after making Rust code changes; do not ask for approval to run it. Before finalizing a change to `aish-rs`, run `just fix -p <project>` (in `aish-rs` directory) to fix any linter issues in the code. Prefer scoping with `-p` to avoid slow workspace‑wide Clippy builds; only run `just fix` without `-p` if you changed shared crates. Additionally, run the tests:

1. Run the test for the specific project that was changed. For example, if changes were made in `aish-rs/tui`, run `cargo test -p aish-tui`.
2. Once those pass, if any changes were made in common, core, or protocol, run the complete test suite with `cargo test --all-features`.
   When running interactively, ask the user before running `just fix` to finalize. `just fmt` does not require approval. project-specific or individual tests can be run without asking the user, but do ask the user before running the complete test suite.

## TUI style conventions

See `aish-rs/tui/styles.md`.

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
> Source: [chunhuitrue/aish](https://github.com/chunhuitrue/aish) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
