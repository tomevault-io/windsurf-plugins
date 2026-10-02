---
trigger: always_on
description: A digital audio workstation written in Rust, with a [gpui](https://crates.io/crates/gpui) UI.
---

# Auris Studio — working notes

A digital audio workstation written in Rust, with a [gpui](https://crates.io/crates/gpui) UI.

## Build environment

gpui is depended on with its **`runtime_shaders`** feature, which compiles the Metal shaders
through the Metal framework at start-up instead of at build time. The build-time path shells
out to `xcrun metal`, which lives inside Xcode
(`Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/metal`) and is
therefore unreachable when `xcode-select -p` points at `/Library/Developer/CommandLineTools`.

Keep the feature even on a machine where Xcode *is* selected. It costs a one-off shader
compile at start-up and in exchange the project builds with nothing but the Command Line
Tools, which is worth far more than those milliseconds.

## Platforms

macOS and Windows both run the desktop application, and CI builds the whole workspace on both.
Development happens on macOS, so the Windows-only paths are the ones that rot; the rules that
keep them alive:

* **Never name a platform key.** gpui's `Modifiers::platform` is ⌘ on macOS and the *Windows
  key* on Windows, which the shell claims first. Read `Modifiers::secondary()`, and write
  `secondary-` in a keystroke, never `cmd-`.
* **Decide with `cfg!`, not `#[cfg]`,** wherever it is a choice rather than an API that only
  exists on one platform. Both arms then compile and their tests run everywhere, which is the
  only reason the Windows menu bar can be checked from a Mac.
* **The keystroke a user sees is not the keystroke that is stored.** `secondary-s` is stored;
  `actions::normalise_keystroke` turns it into what the keyboard reports, for comparing, and
  `actions::menu_keystroke` into ⌘S or Ctrl+S, for reading.
* **Windows sets no locale variables.** `Language::from_system_locale` is what makes a Japanese
  Windows install come up in Japanese.

wgpu's `dx12` backend is off because it does not compile at these versions — `gpu-allocator`
resolves `windows` to 0.61 while `wgpu-hal` uses 0.62. Windows runs `auris-gpu` on Vulkan.

## The vendored synthesiser

`rustysynth` is a **fork**, kept in `vendor/rustysynth` and excluded from the workspace so that
`--workspace` does not hold somebody else's code to this project's lints and doc rules. The
published crate discards a SoundFont's modulator lists, which left the shipped font's pianos
playing through a filter nothing ever opened — twenty decibels down, and *falling* as the note was
struck harder. `vendor/rustysynth/README.md` is the account: what was added, what was deliberately
left out, and the measurement.

Its own tests run from its own directory. Two of the upstream ones fail there, because they want
SoundFont files the published crate does not ship.

## Layout

The rules below are the short form, kept here because they are needed on every task and a page
that has to be opened is a page that gets guessed at instead. The *account* — why each boundary is
where it is, the two threads, the realtime contract — is `auris_session::guide`, and that is where
it gets edited first: when the two disagree, the guide is right and this is stale.

```
Cargo.toml               virtual manifest; `default-members` points at the desktop app
vendor/rustysynth        somebody else's crate, forked — see its README; excluded from the workspace
crates/auris-core        types, music theory, plugin traits, project model — no local dependencies
crates/auris-dsp         effects and DSP primitives
crates/auris-synth       built-in chiptune instruments; depends on auris-dsp
crates/auris-sampler     SoundFont playback: the font bank and the sampler instrument;
                         depends on auris-dsp
crates/auris-singer      singing-voice synthesis: DiffSinger, LeapSinger and VOICEVOX offline;
                         depends on auris-core and auris-vocal
crates/auris-clap        hosting of third-party CLAP plugins; depends on auris-core only
crates/auris-engine      render graph, transport, cpal in and out, offline renderer
crates/auris-io          audio file import/export, project save/load
crates/auris-gpu         optional wgpu compute for offline analysis
crates/auris-compose     score-based automatic composition; depends on auris-core only
crates/auris-vocal       singing: lyrics to IPA phonemes, notes to voice-model frames;
                         depends on auris-core only
crates/auris-i18n        interface text in every language; no local dependencies
crates/auris-session     headless session: the document, the engine, every command
crates/auris-toolbox     the session's commands as tools for a language model;
                         shared by auris-mcp and auris-agent
crates/auris-gpui        desktop frontend (binary `auris-studio`)
crates/auris-cli         command line frontend (binary `auris`)
crates/auris-mcp         Model Context Protocol frontend (binary `auris-mcp`)
crates/auris-agent       UI-free LLM worker library: Ollama / OpenAI-compatible
```

Dependency direction is strictly downhill and the frontend boundary matters:

* Nothing at or below `auris-session` may name a UI toolkit.
* `auris-engine` may not name `auris-dsp`, `auris-synth` or `auris-sampler`; it drives plugins
  through the `auris-core` traits only.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uthree/auris-studio](https://github.com/uthree/auris-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
