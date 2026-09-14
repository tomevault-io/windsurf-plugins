---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
cargo run                                   # run the TUI
cargo test                                  # all unit tests
cargo test <name_substring>                 # run a single test by name fragment
cargo test app::tests                       # run one module's tests
cargo clippy --all-targets -- -D warnings   # lint (warnings are errors here)
cargo fmt --check                           # formatting gate
cargo doc --no-deps --lib                   # intra-doc links must resolve
cargo build && scripts/smoke.sh             # drive the real binary in tmux (every phase, in parallel)
scripts/smoke.sh 55 permission              # only some phases (an id, a range, a name substring)
scripts/smoke.sh --list                     # the phases, their tags and last durations
bash scripts/smoke/phases/055-permission.sh # one phase on its own, output live (docs/smoke.md)
scripts/release.sh check [vX.Y.Z]           # Cargo.toml, Cargo.lock, the README badge and CHANGELOG.md agree (docs/release.md)
scripts/release.sh build [TARGET]           # one platform's release archive + .sha256 into dist/
scripts/release.sh verify dist              # the archives: checksums, layout, CPU, `alter-zero --version`
scripts/release.sh notes X.Y.Z              # the release notes the workflow publishes: CHANGELOG.md's section, nothing else
scripts/release.sh selftest                 # the release tooling's own fixture-driven tests
scripts/release.sh prepare X.Y.Z            # bump the version everywhere, roll [Unreleased] into a dated section, then tag
cargo run --release --example mem_probe     # /model parse RSS (docs/memory.md)
cargo build --release --timings && scripts/build_timings.py   # where a release build's time goes (docs/build-time.md)
DISPLAY=:99 cargo test --test clipboard_linux -- --ignored   # the X11 paste read, under Xvfb
(cd telemetry && node --test)               # the telemetry collector's pure half (docs/telemetry.md)
```

The standard pre-commit gate used throughout this project is: `cargo fmt --check`
+ `cargo clippy --all-targets -- -D warnings` + `cargo test` + `cargo doc
--no-deps --lib` all clean. The doc build is part of the gate because the crate
denies warnings, which promotes a broken intra-doc link to an error: a public
item's docs may not link to a private one, so moving an item between modules (or
narrowing its visibility) breaks the links that pointed at it. When that happens
the fix is to qualify the path if the target is still public
(`[`x`]` → `` [`x`](App::x) ``), else demote the link to a plain code span
(`[`x`]` → `` `x` ``) so the prose still names it.

CI runs that gate on every push to `main` and every pull request
(`.github/workflows/ci.yml`: the gate, the smoke suite under tmux, the release
tooling's `selftest` + `check`, and the telemetry collector's `node --test`),
and a `vX.Y.Z` tag push runs `.github/workflows/release.yml` — the same gate,
then one release build per platform (Linux x86_64 and arm64, macOS Intel and
Apple silicon) packaged, checksummed and verified by `scripts/release.sh`, and
a GitHub release whose notes are `CHANGELOG.md`'s section for the version
(`docs/release.md`). Cutting a release is `scripts/release.sh prepare X.Y.Z`,
a commit, an annotated tag and a push; `workflow_dispatch` rehearses the whole
pipeline without publishing, and every step runs locally the same way. Users
install a release with the one-line `install.sh` (`curl … | sh`, POSIX `sh`,
checksum-verified), which the selftest drives against
`scripts/release/release_server.py`, a stand-in for github.com's release pages.

Toolchain: Rust **edition 2024**, `ratatui = 0.30.1` (crossterm is re-exported as
`ratatui::crossterm` — import it from there, not as a separate crate), plus
`ratatui-image` for the inline pictures (`default-features = false`: its
default `chafa-dyn` links a C library we don't have, and `image-defaults`
would turn on every `image` codec — see `docs/images.md`), `png` reached
directly for its row-streaming decoder (`images::fitted`, `docs/memory.md`),
and — Linux only — `x11rb` + `wl-clipboard-rs`, arboard's own backends at
arboard's own versions, for the clipboard's `image/png` bytes
(`clipboard::linux`, `docs/image-paste.md`),
`unicode-width` for display-width math, `unicode-segmentation` for the textarea's
grapheme-aware cursor/wrapping, and **`tokio`** (current-thread runtime) +
`tokio-stream` for the async event loop. The `Cargo.toml` `crossterm` entry exists
*only* to enable its `event-stream` feature (for `EventStream`); code still imports
crossterm through `ratatui::crossterm`, never as `crossterm::…`. `rust-toolchain.toml`
pins the toolchain; a `[lints]` table in `Cargo.toml` bakes the gate into every
build (`unsafe_code = "forbid"`, plus `warnings` and `clippy::all` denied).
Dependency features are cut to what the code reaches — `syntect`/`two-face`
load only two-face's prebuilt onig dumps (`parsing` + `regex-onig` /
`syntect-onig`, never the `plist`/`yaml` file loaders `default-onig` carried),
`ratatui` runs without `all-widgets`/`macros` (the calendar widget was the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linuztx/alter-zero](https://github.com/linuztx/alter-zero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-14 -->
