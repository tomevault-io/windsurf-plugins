---
trigger: always_on
description: **The product is one Rust binary. There is no Node in it, and none next to it
---

# Agent notes

## ⛔ READ THIS FIRST — what is vestigial, and what is not

**The product is one Rust binary. There is no Node in it, and none next to it
either.** The `npm/` layer, the `Makefile`, `.github/`, the `scripts/*.test.mjs`
suite, `scripts/real-agent-e2e.py` and `scripts/check-release-tag.mjs` are
deleted, not dormant. If a commit message, an old note here, or your own
recollection points at one of them, it is stale.

There is no plugin host, no profile system, no client-plugin tree, no
`ctx.acpClient`, no agent pool and no second process. Nothing answers the
`_crow/tui/*` extension family in `src/ext.rs` either, which is why four slash
commands are parked (below). The user runs `./target/release/crow` directly.

What *is* still vestigial, and exactly where:

- `src/theme.rs` is the palette layer and nothing else: the house ramp
  `CROW_50…CROW_900`, the six `BUILTIN_PALETTE_IDS` packs (`default` wears the
  house ramp; four Catppuccin flavors; one retained blue pack, listed last
  because it is the odd one out), and the protocol-0 parse. There is no logo
  module and no lockup asset anywhere in the tree.
- `/plugins`, `/dynamic-plugins`, `/ui` and `/liang` are **parked, not live**:
  commented out of `SLASH_COMMANDS` at their alphabetical positions, each with a
  note naming the machinery that is still compiled and tested. `src/ext.rs` is
  the wire-name table for the host that does not exist; `src/slots.rs` and
  `src/pet.rs` are the receiving ends. `run_slash` still resolves all four.
- **Anything that "arrives from a compositor slot snapshot" never arrives.**
  `app.harness_badge` is fed from the `conversation.harness` slot in
  `src/app/pump.rs`, so it is permanently empty. Chrome, badges and palettes
  must be driven from the Rust side (`CtlEvent` / `AppEvent`), never from a
  host that does not exist.
- Every builtin slash command still needs an explicit zh description in
  `src/locale.rs command_desc`, or
  `every_builtin_command_has_an_explicit_zh_desc` fails. A real gate on a
  vestigial requirement — do not "clean it up" without also removing the test.

Consequences that have already bitten an agent:

- **The `/` namespace has three sources, and builtins win.** `SLASH_COMMANDS`
  in `src/app/slash_catalog.rs` is resolved client-side in the painter. The
  agent's `availableCommands` arrive as host skills in the same menu
  (`skills_from_available_commands` at `events.rs:950`, fed from
  `acp.rs:1804-1819` and `acp/v2.rs:785`) — that path is live, and this
  repo's own agent uses it (`src/crow_cli/agent/main.py:566`). A compositor
  could also push client plugin commands over
  `_crow/tui/commands/update`; nothing does, so that source is parked with the
  four commands above. Both non-builtin sources are filtered against the
  builtin names (`app/slash.rs:43,74`), so a builtin can never be shadowed,
  and each group stays alphabetical.
- **The Rust suite is the only gate.** There is no other suite, and no CI.

Read [`README.md`](README.md) for what this repo is becoming versus what it is
today. [`docs/README.md`](docs/README.md) indexes what is left in `docs/`.

## Commands & verification

- **Rust is the only gate**: `cargo test --locked` and `cargo check --locked --tests`. `scripts/cargo-guard.sh` wraps cargo with cache-size and disk guards (`CROW_RUST_CACHE_MAX_GIB`); there is no Makefile, so call it directly or just run cargo. No clippy/fmt gate exists. Never run repo-wide `cargo fmt` — `acp.rs`, `app.rs` and `ui.rs` carry hundreds of pre-existing rustfmt markers; format only files that are entirely new, and check `rustfmt --check` line numbers against `git diff -U0` for existing ones.
- ⛔ `scripts/cargo-guard.sh` must **never** auto-clean the target dir. It used to `cargo clean` `$CROW_CARGO_TARGET_DIR` (= `$PWD/target`) at 20 GiB and deleted a release build mid-test-run. Over the limit it warns; pruning is explicit (`cargo-guard.sh prune`). The user builds with bare `cargo build --release -j 6` into `$PWD/target` and tests from it. Do not reintroduce auto-clean.
- Linker OOM: on small-RAM machines `cc`/`ld` can get killed (signal 9) linking the test binary. Retry with `RUSTFLAGS="-C link-arg=-fuse-ld=mold" cargo test` (mold is much lighter). `release` uses `lto = true` and is slow — verify with debug builds.
- Rust unit tests live in `tests/unit/*__tests.rs` but are wired in by a `#[cfg(test)] #[path = "../tests/unit/…"] mod tests;` include at the bottom of the owning `src/*.rs` file — add that include for new test files.
- No-TTY visual check: `cargo run -- --dump-frame 100x34` renders the canned demo frame as text; useful to diff transcript/markdown output before and after a rendering change.
- Demos: `crow --demo` runs the canned transcript on the built-in `default` theme. `--demo-skin` is gone and `tests/cli_help.rs` pins its absence — do not reintroduce it. The full flag list is `parse_args_from` in `src/main.rs`; nothing in the binary spawns `node`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crow-cli/crow-cli](https://github.com/crow-cli/crow-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
