---
trigger: always_on
description: When you need the user to interact with an external service (Slack, GitHub, cloud providers, etc.), always lead with the lowest-friction option. Do not give step-by-step UI walkthroughs when a single copy-paste or CLI command exists.
---

# User interaction principles

## Minimize user friction for external services

When you need the user to interact with an external service (Slack, GitHub, cloud providers, etc.), always lead with the lowest-friction option. Do not give step-by-step UI walkthroughs when a single copy-paste or CLI command exists.

Rules:

1. **Open URLs in their browser** — use `open <url>` (macOS) or `xdg-open <url>` (Linux). Their default browser is already authenticated to most services, so no login needed.
2. **Provide manifests / config files** that can be pasted in one shot, not step-by-step click instructions.
3. **Check for CLI tools first** — many services have CLIs (`slack`, `gh`, `gcloud`, `aws`) that can do setup from the terminal with zero browser interaction. Run `which <tool>` before falling back to browser-based flows.
4. **Never give more than 4-5 user-facing steps.** If you're writing more, you're not automating enough.
5. **Do all local work silently** — write config files, set permissions, wire integrations. Only present what the user must do manually.
6. **Pre-fill where possible** — use URL query parameters, manifest templates, and pre-configured defaults to skip manual entry.

Bad example: "Go to slack.com, click Create App, click From Scratch, enter a name, pick workspace, click Create, navigate to Incoming Webhooks, toggle on, click Add, pick channel, click Allow, copy URL, paste here" (12 steps)

Good example: "I'll open the Slack app setup page in your browser. Paste this manifest: [yaml]. Click Create, then Add Webhook, pick your channel, and paste the URL here." (4 steps + browser auto-opens)

---

# Rust/codex-rs

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
  - Do not add these comments for string or char literals unless the comment adds real clarity; those literals are intentionally exempt from the lint.
  - If you add one of these comments, the parameter name must exactly match the callee signature.
- When possible, make `match` statements exhaustive and avoid wildcard arms.
- When writing tests, prefer comparing the equality of entire objects over fields one by one.
- When making a change that adds or changes an API, ensure that the documentation in the `docs/` folder is up to date if applicable.
- If you change `ConfigToml` or nested config types, run `just write-config-schema` to update `codex-rs/core/config.schema.json`.
- If you change Rust dependencies (`Cargo.toml` or `Cargo.lock`), run `just bazel-lock-update` from the
  repo root to refresh `MODULE.bazel.lock`, and include that lockfile update in the same change.
- After dependency changes, run `just bazel-lock-check` from the repo root so lockfile drift is caught
  locally before CI.
- Bazel does not automatically make source-tree files available to compile-time Rust file access. If
  you add `include_str!`, `include_bytes!`, `sqlx::migrate!`, or similar build-time file or
  directory reads, update the crate's `BUILD.bazel` (`compile_data`, `build_script_data`, or test
  data) or Bazel may fail even when Cargo passes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Agents2AgentsAI/ata](https://github.com/Agents2AgentsAI/ata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
