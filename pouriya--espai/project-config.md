---
trigger: always_on
description: Coding conventions for this project. Follow these when writing or modifying code.
---

# Agent Guidelines

Coding conventions for this project. Follow these when writing or modifying code.

## Context: kept plain

The whole native host is one [`main.rs`](src/main.rs) with two
dependencies (`serde_json`, `tiny_http`), and the extension is six files the browser loads
as they are — no bundler, no npm, no build step beyond copying them next to the manifest
rendered from the template in [`extension/`](extension/). Before
adding a dependency or splitting a module, be sure the thing you are adding pays for the
loss of "read one file, understand the system". `main.rs` stays one file; if it passes
roughly 1200 lines, that is the moment to reconsider, not before. Count the body above
`#[cfg(test)]` — the test module costs a reader nothing. **That body is at ~1194 lines
today, so the next feature of any size is the one that triggers this.** Splitting is a
decision to raise with whoever owns the project, not one to take while doing something
else.

The style rules below favour locally-readable code over idiomatic density. They are
choices, not oversights — do not "modernize" them into iterator chains, combinator
pipelines, or extracted abstractions.

## Generic primitives, never site adapters

espai must not grow a `twitter.js` or an `instagram.js`. Those sites regenerate their
markup continuously and every selector committed here would be broken within weeks, with
nothing to warn us but a user reporting that likes silently stopped working. The three
capabilities that make an app site tractable — a snapshot with refs, trusted input, and the
network capture — are all site-agnostic, and together they are enough. Anything
site-shaped belongs in the agent's prompt, where it costs nothing to be wrong.

## Never write to stdout

**`println!` is a fatal bug.** stdout is the native messaging channel, and one stray byte
desynchronises the length-prefixed frame stream: Chrome sees a garbage length, drops the
port, and kills this process. There is no recovery and the symptom is a silent
disconnection, not an error.

Every diagnostic goes to stderr via `eprintln!`, prefixed `espai:`. Chrome inherits our
stderr from the browser process, so it lands wherever Chrome was started from — the
terminal if launched from one, nowhere if launched from the desktop. Treat it as best
effort. Nothing may depend on a human seeing it.

## How Chrome starts and kills this process

Chrome spawns one host process **per `connectNative()` call** — not per tab, not per
browser. It is lazy: nothing runs until the service worker connects. `argv[1]` is the
calling extension's origin. The working directory is unspecified, so never use a relative
path.

Chrome, Chromium, Edge and Brave each read the host manifest from **their own profile
root**, and a manifest in the wrong one is silently ignored — the symptom is an extension
that loads fine, reports no error a user would notice, and never starts the host.
`make chrome` and `make chromium` write a ready `com.espai.host.json` beside each built
extension; `make install-chrome` and `make install-chromium` copy it into the root that
browser actually reads.

Those install targets **fail on a profile root that does not exist** rather than creating
one, and that is not a convenience worth adding later. Creating `~/.config/google-chrome` on
a machine that only runs Chromium leaves a manifest nothing will ever read and a directory
that looks installed — the exact failure the check exists to catch. Edge and Brave have no
make target for the same reason: a root this repo has to guess is a root it can guess wrong.
[`install.sh`](install.sh) writes all four, but it never guesses either — it copies into
every root that already exists and creates none.

Every manifest points at `bin/espai`, which `release` copies out of `target/`. A path into
`target/` would break on the next `cargo clean` or debug build, and break silently, since a
missing host binary looks exactly like a missing manifest.

The two halves version together: `version` in [`Cargo.toml`](Cargo.toml) and in
[`manifest.chromium.json.in`](extension/manifest.chromium.json.in) name the same release
and are bumped in the same commit. They ship as one thing and speak a private frame
protocol to each other, so a version that identifies the extension but not the host binary
behind it describes half a system. `make check-version` enforces the pairing and runs as part
of `make all`. Note that Chrome refuses to load an extension whose version goes backwards, so
the manifest only ever moves forward.

Shutdown arrives as **EOF on stdin**, and `UnexpectedEof` on the length read is therefore
the ordinary exit path, not an error to log loudly. The host deregisters and exits; a host
that ignored EOF would be killed anyway.

## Why there is a heartbeat

The MV3 service worker that owns our port is killed after ~30s idle, and killing it closes
the port and kills this process. Native port traffic resets Chrome's idle timer, so the
20s ping in `main` is the only thing keeping espai reachable between agent calls. It looks
like a no-op — replies to `id: 0` are discarded — and deleting it appears to change
nothing, until the eye is dead every time an agent actually reaches for it.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pouriya/espai](https://github.com/pouriya/espai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-08 -->
