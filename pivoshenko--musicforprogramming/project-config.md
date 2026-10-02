---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

A terminal player for [musicforprogramming.net](https://musicforprogramming.net), written in Rust. A
three-crate cargo workspace: `mfp-core` (shared model, wire protocol, config, paths), `mfp-daemon`
(the `mfp-daemon` binary), `mfp-tui` (the `mfp` binary - both the interface and the scriptable CLI).

macOS and Linux only. The client reaches the daemon over a Unix domain socket and the state directory
is chmodded through `PermissionsExt`, so there is no Windows target and CI has no Windows leg.
`AGENTS.md` is a symlink to this file.

`plugins/herdr/mfp.player` is a Herdr plugin, not Rust: a `herdr-plugin.toml` manifest over a few
scripts in `plugin/`. Each action runs an `mfp` subcommand and then reports the state it left behind,
because a plugin command's stdout goes to Herdr's command log rather than the screen. The pane starts
the daemon through a throwaway client before launching the interface, so the daemon is never a child
of a pane somebody dismisses. It is outside the workspace, so `just check` does not see it - changing
a CLI verb or an exit code is what would break it.

## Commands

`just` is the task runner - `just --list` for the full set, or the recipe table in `CONTRIBUTING.md`.
`just check` is `lint` + `audit` + `test` + `build`, and CI runs those same recipes, so a green
`just check` locally means a green CI.

```bash
cargo test -p mfp-core                  # one crate
cargo test seek                         # by test-name substring
cargo test -- --nocapture               # keep stdout
cargo test -p mfp-daemon -- --ignored   # the tests skipped by default
```

The default run is hermetic - no network, no audio device - so a failure is a real failure rather
than a sandbox artifact. Anything that needs the real world is `#[ignore]`d with its reason in the
attribute: an audio output device, the network, or a benchmark. `mfp-daemon/tests/end_to_end.rs`
spawns the binary and is audible, and the player has no volume of its own.

`just generate-changelog` regenerates `CHANGELOG.md` from the commit history with git-cliff.
`just generate-social-preview` rasterizes `assets/preview_social_dark.svg` into the 1280x640 PNG for
GitHub's social preview; that SVG's colours come from `crates/mfp-tui/src/ui/theme.rs`, so a palette
change belongs in both.

## Architecture

**The split.** The daemon is the sole authority over playback and download state and outlives every
client - closing a pane must not interrupt audio, which is why `ctrl+c` in the interface only
detaches and `q` is the one key that shuts the daemon down. It exits only on an explicit
`shutdown` or the configured `idle_timeout_secs` (unset by default, meaning never). `mfp`
autostarts `mfp-daemon` when nothing is listening on the socket, looking beside itself and then on
`PATH`; `$MFP_DAEMON` overrides.

**Concurrency.** `rodio` blocks and wants an OS thread of its own, while the socket server and the
downloads want `tokio`. `audio/` runs on one dedicated thread consuming a command channel and never
awaits; everything else runs on the runtime and never blocks on audio. They meet at
`state::SharedState` (an `Arc<Mutex<StateSnapshot>>`), with `StateStore` (`state.json`) as the durable
layer beneath it. `state::lock` recovers from poisoning rather than propagating it.

**Transport.** `ipc/` is a Unix domain socket speaking newline-delimited JSON: a client writes
`Request` lines and reads `Frame` lines, each either a `Response` echoing a request's `id` or an
unsolicited `EventFrame` carrying a whole `StateSnapshot`, never a delta. Pushes are level-triggered
on a poll of the shared state, not edge-triggered per change, so socket traffic stays bounded however
chatty playback becomes. `mfp-tui/src/client.rs` is a synchronous `UnixStream` with read timeouts on
purpose - the interface is a blocking `ratatui` loop, not a second async runtime.

**Dispatch seam.** `ipc/server.rs` defines the `Player` trait and dispatches against it rather than
against the engine, so framing, ordering, error codes, and the lifecycle are testable over a real
socket with no audio device. Every method returns as soon as a command is *accepted*; clients learn
the consequence from the next snapshot.

**Catalog.** `catalog/` resolves `feed` (authoritative, a failure fails the catalog) then `enrich`
(best-effort from the site's client bundle, the only source of track listings; every failure is
logged at debug and skipped), behind the disk cache in `cache` with a 6-hour TTL. A stale cache is
served when a refresh fails. This is what lets the player start with no network.

**Analyser.** `audio/spectrum.rs` reproduces Web Audio's `getByteFrequencyData` - mono downmix, Hann
window, 2048-point transform, exponential smoothing against the previous frame, linear scaling from
`SPECTRUM_MIN_DB` to `SPECTRUM_MAX_DB` onto `0..=255` - so a client can run the site's own analyser
arithmetic unchanged.

**Updates.** `mfp-tui/src/update/` is the only part of the client that reaches the network, and it
reaches GitHub rather than the site. `mod.rs` resolves the latest release and decides whether this

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pivoshenko/musicforprogramming](https://github.com/pivoshenko/musicforprogramming) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
