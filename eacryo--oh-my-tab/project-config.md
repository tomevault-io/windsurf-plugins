---
trigger: always_on
description: Guidance for agents working in this repository.
---

# AGENTS.md

Guidance for agents working in this repository.

## Project

`oh-my-tab` is a macOS menu-bar window switcher. It intercepts Command+Tab or Option+Tab, displays a floating application/window overlay, and raises the selected window through Accessibility APIs. Optional modules provide mouse enhancement and clipboard history.

The app is Rust-only and calls AppKit, CoreGraphics, and ApplicationServices through `objc2` FFI; there is no Swift bridge or Rust UI framework.

## Build, run, and verification

```sh
cargo fmt
cargo check
cargo clippy
cargo test
scripts/dev-restart.sh
```

- The normal `cargo test` suite is headless-safe; GUI/permission-dependent smoke tests are marked `#[ignore]`.
- Documentation, comments, localization, and `AGENTS.md` changes need no Rust tests. Rust changes require `cargo fmt`; behavioral changes require targeted tests, plus `cargo check` when interfaces or compilation are affected.
- Cross-module, unsafe/FFI, concurrency, configuration, build, or release changes require the full gate above. Before handing off a completed feature, always run the full gate and keep all checks clean.
- Start the app with `scripts/dev-restart.sh`, never directly with `cargo run`. For runtime changes, run it after the full gate. If it reports `restart FAILED`, inspect the newest log under `~/Library/Logs/oh-my-tab/`, diagnose, and retry.
- `scripts/dev-restart.sh` defaults to a **debug** build (`cargo build`): fast iteration with every debug assertion on (objc2 `msg_send!` signature verification, `debug_assert_main_thread`, overflow checks). Use it for functional iteration and handoff.
- For feel/perf validation (scrolling, animation, latency), run `scripts/dev-restart.sh --opt`. It uses the `dev-opt` cargo profile (`target/dev-opt/`): optimized like release while keeping `debug-assertions`, so runtime speed is representative but development still fails fast. Neither mode changes the dev bundle identity (`com.eacryo.oh-my-tab.dev`); both write `dist/Oh-My-Tab-Dev.app`, which `scripts/release-dev.sh` also uses for the dev channel.
- Report the timestamp-based `build-version` printed by the script.
- Pass development switches through the script instead of editing code: any `--flag[=value]` argument the script does not own is forwarded to the app (and `scripts/dev-restart.sh -- <args>` forwards argv verbatim), so a new switch needs no script edit. Switches apply to that launch only — the script pkills old instances first. Use this to reach states that are otherwise hard to reproduce (first-run onboarding, permission branches, update notices); a feature that only appears in such a state should expose a `--`-style switch for verification, and it is parsed through `crate::dev_flags`.
- **No environment variable is read, forwarded, or echoed anywhere in the launch chain.** The app runs under launchd, which does not inherit the caller's shell environment, so switches ride argv only; `crate::dev_flags` (Rust) and the script's argv passthrough are the only channels. Do not reintroduce an environment switch, dump the environment, or print a variable's value: the caller's shell holds cloud credentials and proxies, and one leak is an incident (a missing forward filter leaked the whole environment once, on 2026-09-22).

## Testing tiers

Three layers, split by **who decides pass/fail** — not by which transport drives them (`cua-driver call` and the MCP tools are the same tool set on the same daemon; the CLI is the scriptable face, MCP the agent-facing one).

| Tier | Decides | How | Repeatable | Gate |
| --- | --- | --- | --- | --- |
| A1 | script | `cargo test` plus the `--smoke-*` runners (real AppKit view tree, headless-safe) | yes | yes, every change |
| A2 | script | `scripts/e2e/*.sh`, run through `scripts/e2e/run-all.sh` (the entry point; scenarios that steal focus are skipped unless `--include-focus`), driven by the `cua-driver` CLI and asserting app-written JSON state (`--e2e-state=<path>`) plus real WindowServer state | yes | before handoff/release |
| B | agent or person | MCP tools plus screenshots: taste, first-pass UI review, failure triage, bug investigation | no | never |

- Anything assertable belongs in A. Big problems usually are (state and OS facts), small ones often are not (visual taste) — severity is not the split, assertability is.
- A2 covers what only real input reaches: the global hotkey → summon → raise path, permission branches, cross-app behavior. `--e2e-state` exists because AX cannot express CALayer content (the sidebar highlight pill) or internal state (the selected card index), so the app states those facts itself instead of the test guessing from pixels. A quick hotkey press-release legitimately skips the display path; display and layout assertions belong to A1 smoke runners.
- **Promotion rule: every bug found in tier B must land an assertion in tier A** — or, when it cannot be asserted yet, a state field that makes it assertable. Otherwise the same regression returns unnoticed.
- A2 hotkey scenarios press through the system event stream and therefore steal focus: run them only with the user's consent, never in a background loop.

## Architecture and invariants


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eacryo/oh-my-tab](https://github.com/eacryo/oh-my-tab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
