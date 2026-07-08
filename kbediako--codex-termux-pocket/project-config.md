---
trigger: always_on
description: When writing complex features, long-running tasks, or significant refactors, use an ExecPlan as described in `PLANS.md`. If the user asks for an ExecPlan (or the work is clearly multi‑step and risky), create or update the ExecPlan and follow it from design through implementation, keeping it current as decisions and progress change.
---

# Rust/codex-rs

## ExecPlans

When writing complex features, long-running tasks, or significant refactors, use an ExecPlan as described in `PLANS.md`. If the user asks for an ExecPlan (or the work is clearly multi‑step and risky), create or update the ExecPlan and follow it from design through implementation, keeping it current as decisions and progress change.

If this repo lacks `PLANS.md` or an ExecPlans section in `AGENTS.md`, add them by copying `~/.codex/PLANS.md` and noting the change. `~/.codex/PLANS.md` is the global template only; if the user requests a cross-repo/global plan, use `~/.codex/EXEC_PLAN.md`.

## Termux Fork Notes

- Prefer `codex-update-alpha` for Android/Termux alpha maintenance; do not default to ad-hoc local Cargo rebuilds.
- The supported update order on Termux is: upstream ARM64 musl artifact, then fork `remote-artifact`, then explicit source retry with `CODEX_TERMUX_ALLOW_SOURCE_FALLBACK=1`.
- If helper or workflow commits are added on this fork, update `scripts/termux/patch_audit.tsv` in the same change so `auto` mode can classify them correctly.
- Keep the main `README.md` short. Put operational detail in `docs/termux-mobile-update.md`.

In the codex-rs folder where the rust code lives:

- Crate names are prefixed with `codex-`. For example, the `core` folder's crate is named `codex-core`
- When using format! and you can inline variables into {}, always do that.
- Install any commands the repo relies on (for example `just`, `rg`, or `cargo-insta`) if they aren't already available before running instructions here.
- Never add or modify any code related to `CODEX_SANDBOX_NETWORK_DISABLED_ENV_VAR` or `CODEX_SANDBOX_ENV_VAR`.
  - You operate in a sandbox where `CODEX_SANDBOX_NETWORK_DISABLED=1` will be set whenever you use the `shell` tool. Any existing code that uses `CODEX_SANDBOX_NETWORK_DISABLED_ENV_VAR` was authored with this fact in mind. It is often used to early exit out of tests that the author knew you would not be able to run given your sandbox limitations.
  - Similarly, when you spawn a process using Seatbelt (`/usr/bin/sandbox-exec`), `CODEX_SANDBOX=seatbelt` will be set on the child process. Integration tests that want to run Seatbelt themselves cannot be run under Seatbelt, so checks for `CODEX_SANDBOX=seatbelt` are also often used to early exit out of tests, as appropriate.
- Always collapse if statements per https://rust-lang.github.io/rust-clippy/master/index.html#collapsible_if
- Always inline format! args when possible per https://rust-lang.github.io/rust-clippy/master/index.html#uninlined_format_args
- Use method references over closures when possible per https://rust-lang.github.io/rust-clippy/master/index.html#redundant_closure_for_method_calls
- Avoid bool or ambiguous `Option` parameters that force callers to write hard-to-read code such as `foo(false)` or `bar(None)`. Prefer enums, named methods, newtypes, or other idiomatic Rust API shapes when they keep the callsite self-documenting.
- When you cannot make that API change and still need a small positional-literal callsite in Rust, follow the `argument_comment_lint` convention:
  - Use an exact `/*param_name*/` comment before opaque literal arguments such as `None`, booleans, and numeric literals when passing them by position.
  - A method's sole non-self argument is exempt when the method and parameter names match, such as `.enabled(false)` for `fn enabled(&self, enabled: bool)`.
  - Do not add these comments for string or char literals unless the comment adds real clarity; those literals are intentionally exempt from the lint.
  - The parameter name in the comment must exactly match the callee signature.
  - You can run `just argument-comment-lint` to run the lint check locally. This is powered by Bazel, so running it the first time can be slow if Bazel is not warmed up, though incremental invocations should take <15s. Most of the time, it is best to update the PR and let CI take responsibility for checking this (or run it asynchronously in the background after submitting the PR). Note CI checks all three platforms, which the local run does not.
- When possible, make `match` statements exhaustive and avoid wildcard arms.
- Newly added traits should include doc comments that explain their role and how implementations are expected to use them.
- Discourage both `#[async_trait]` and `#[allow(async_fn_in_trait)]` in Rust traits.
  - Prefer native RPITIT trait methods with explicit `Send` bounds on the returned future, as in `3c7f013f9735` / `#16630`.
  - Preferred trait shape:
    `fn foo(&self, ...) -> impl std::future::Future<Output = T> + Send;`
  - Implementations may still use `async fn foo(&self, ...) -> T` when they satisfy that contract.
  - Do not use `#[allow(async_fn_in_trait)]` as a shortcut around spelling the future contract explicitly.
- When writing tests, prefer comparing the equality of entire objects over fields one by one.
- Do not add tests for values that are statically defined.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kbediako/codex-termux-pocket](https://github.com/Kbediako/codex-termux-pocket) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
