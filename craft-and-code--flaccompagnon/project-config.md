---
trigger: always_on
description: A cross-platform desktop app (Rust + Tauri v2) that detects fake "lossless" audio. This file is the contract for how code in this repository is written. Read it before making changes.
---

# FlacCompagnon — working conventions

A cross-platform desktop app (Rust + Tauri v2) that detects fake "lossless" audio. This file is the contract for how code in this repository is written. Read it before making changes.

## Language

- **Code, comments, commit messages, UI strings: English.**
- **Conversation with the maintainer: French.**

## Architecture

```
core/                     Pure Rust analysis library. No Tauri dependency, fully unit-testable.
  src/decode/             One module per decode path (generic, FLAC+MD5, DSD, playback).
  src/dsd/                DSD container parsing + the spectral heuristics, kept apart.
services/                 Tag editing, conversion, playlists, and relocation.
  src/tags/               Tag read/write; cover art in its own file.
cli/                      Standalone `flaccompagnon` analysis executable.
src-tauri/                The Tauri v2 desktop app wrapping `core` and `services`.
  src/lib.rs              Wiring only: modules, run(), generate_handler!.
  src/commands/           One file per command domain.
  src/lookup/             Online tag lookup, one file per provider.
  src/menu.rs             Native menu bar.
  src/playback.rs         Audio engine (owns the cpal stream on its own thread).
src/                      Frontend: React + TypeScript, built by Vite.
index.html                App shell only — see "index.html" below.
site/                     The GitHub Pages marketing site (unrelated to the app build).
```

The split is deliberate: analysis and report types belong in `core`, while editing and conversion belong in `services`. Both remain verifiable with `cargo test` alone.

## Factor out shared behaviour, not just shared lines

**The second time the same behaviour appears, it becomes one component, one hook, or one function — used by both call sites.** This applies across the whole repo (React components, hooks, `format.ts` helpers, `shared.css` rules, Rust modules) and it is not primarily about file length or typing less. It is about correctness: two copies of the same behaviour are two things that can disagree, and they will, because a fix applied to one is a fix the other never receives.

This rule was written after a real bug. Three drop targets — the cover box, the results list, the conversion panel — each hit-tested incoming drop positions with their own copy of the same rectangle arithmetic. The shared coordinate conversion underneath them was wrong, but the two left-hand, wide targets absorbed the error and kept working, so only the right-hand one appeared broken. Several rounds of debugging went into the one panel that looked at fault, because the duplication hid that the defect was common to all three. One `dropZones.ts` later, there is a single declaration site, a single lookup, and no arithmetic at all. **Duplication doesn't just cost maintenance; it hides where a bug actually lives.**

Practical consequences:

- Two components needing the same visual treatment: the rule goes in `src/shared.css` (`.link-btn`, `.btn`, `.icon-btn`), not copied into each component's stylesheet.
- Two components needing the same markup and interaction: extract a component (`IconButton`, `MarqueeText`) and pass the differences as props.
- Two call sites needing the same pure logic: extract a function into `src/format.ts` or the appropriate Rust module (`isAudioPath`, `convert::pcm`).
- Two places needing the same stateful behaviour: extract a hook (`useLatest`, `useColumnPrefs`).
- The parameters that differ become arguments. If that argument list grows past what one sentence can describe, the two cases were genuinely different after all — that's the signal to stop, not to add a sixth boolean flag.

What this rule is _not_: an instruction to unify things that merely look alike today. Two functions with identical bodies and unrelated reasons to change are better left apart — coupling them means a change to one silently alters the other. The test is whether they would have to change _together_.

## Frontend rules

### One module, one responsibility

- **A component file does one thing.** If you cannot describe a file's job in a single sentence without "and", split it.
- **~200 lines is the target, 300 the hard ceiling** for a component file. Passing it means the component is doing several jobs — split it rather than letting it grow. (A file that is one long static list, e.g. a table of labels, is the exception.)
- A component owns **its own markup, its own local state, and its own event handlers**. Markup for a feature does not live in `index.html` while its logic lives in a module — that separation is by layer, not by responsibility, and it is exactly what this rule exists to prevent.

### State

- Local state stays in the component that uses it (`useState`).
- Shared state is lifted to the nearest common parent and passed down as props. No global mutable singletons.
- Server/backend state (analysis results, tags, covers) is fetched through `src/api.ts` and cached in the owning component, never fetched ad hoc from a leaf component.

### Boundaries


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [craft-and-code/FlacCompagnon](https://github.com/craft-and-code/FlacCompagnon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
