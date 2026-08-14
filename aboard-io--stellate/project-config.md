---
trigger: always_on
description: validates against the live engine vocabulary, MEASURES verifier targets
---

# CLAUDE.md — stellate

A self-contained generative genre-space instrument: a **274-genre**
deterministic vector space (`engine/genre-kernel.js` is the algebra; the anchors
themselves live beside it in `engine/genres-data.js`, incl. real 3/4 odd-meter
anchors — `state.meter`) over one score brain
(`engine/csd-engine.js buildEvents`) with a generative harmony/pipes layer
(`engine/theory.js` + `engine/pipes.js` — docs/MUSIC-MIND.md), **sampled by
default** (full General MIDI via `engine/faust/build/extract-gm.js`, with per-voice
Faust effect chains) and played by a single **Faust WASM engine** (`engine/faust/` — live in the browser and
offline "press" in node), verified symbolically and empirically. It is a worked
example of a generator → verifier → feedback loop: the thing that makes the
music and the thing that checks it live side by side and argue. (Older
references call it "Royal Road vaporwave"; it was renamed **stellate** at
export.)

Faust is the **only** backend on main, which is **csound-free**: no `.csd`, no
`csound` binary anywhere in the toolchain. The entire csound era — `buildCsd`
codegen, `wasm-audio.js`, the `builder.html` song builder, `play.html` player,
the founding `royal-road.csd`, its `render.sh`, the engine A/B tools — is
preserved fully working one `git switch legacy-csound` away
(docs/history/FAUST-PORT.md).

## The one rule

**Source is committed; audio is derived and gitignored.** `engine/csd-engine.js`
(the score brain) / `engine/faust/dsp` (the synthesis) are the capability; every
`.wav`/`.mp3` is regenerable and must never be committed. (The project exists
because we once kept the renders and lost the generator — the founding
`royal-road.csd` — see the README genesis parable. That `.csd` now lives safe on
`legacy-csound`.)

## Where the genre data lives

`engine/genre-kernel.js` is the ALGEBRA — `resolveMulti`/`blend`/`mix`/`track`/
`journey`/`deriveMind`, ~2,700 lines and ~190 KB, all of it code. The inert
literals it used to carry sit in two sibling files, lifted out by a one-time
migration (`tools/build/split-kernel-data.js`, kept as the record of how, not as
something to re-run):

- `engine/genres-data.js` (~5,300 lines, ~645 KB) — `GENRES`, the 274 anchors.
  `Object.keys` order is load-bearing: it drives the confusion-matrix row order
  and the star layout. Append, never reorder.
- `engine/registry-data.js` (~950 lines, ~155 KB) — `SOURCES` / `SOURCE_POOLS` /
  `VOICE_FAMILIES` / `SAMPLES` / `VOXBANK` / `SAMPLERS` / `PERCBANK`: the ids the
  fetch recipes write and the engine resolves.

They are **the source of truth**, not build output. `genres-data.js` is written
both by hand and by the pipeline tools — `genre-tool.js` / `invent-genres.js` /
`rm-genre.js` splice it (and `genre-verifier.js`) through
`/* genre-tool:<name>:genres */` and `:clips` markers.
`registry-data.js` has no automated writer at all: it is edited by hand, by the
sample-CD and found-sound recipes. Both are **classic scripts on purpose, not
JSON-over-fetch**: `engine/genre-kernel.js` merges `window.__GENRES` +
`window.__REGISTRY` at load, and `app/core/state.js` re-exports the kernel as `K`
at module top level, which `app/entries/access.js` (`Object.keys(K.GENRES)`) and
`app/map/layout.js` read synchronously while their modules evaluate. A fetch
would not have resolved yet. So both files load immediately BEFORE
`genre-kernel.js` in `index.html` / `embed.html` / `access.html`, and
`test/gates/boot-smoke.test.js` enforces that order.

`test/gates/kernel-data-identity.test.js` (`verify.sh` row `kerneldata`) holds
them byte-for-byte against HEAD — including key order and float printing, since
every seeded render and the whole matrix are downstream of those exact bytes. It
self-heals on commit, so it flags unintended drift rather than forbidding edits.

**One thing in there is generated and must not be hand-edited: the `info`
blurbs.** `tools/genre/gen-genre-info.js --write` derives all 274 descriptions
from the anchors that ship, so a card cannot promise an instrument the recipe
cannot play (`musicality.checkCardClaims` fails the genre if it does). Every word
comes from a TABLE keyed on an engine value — kit id, synthesis model, sampler
id, bass pattern — never a per-genre string, and there are deliberately no
per-genre overrides. Change the anchor and re-run; the prose follows. Hand-written
blurbs beside live data are what rotted the descriptions the first time.

## Run / test

Browser entry `index.html` at the root; deterministic core + WASM engine in
`engine/` (incl. `engine/faust/`); Node CLIs in `tools/`; gates and harnesses in
`test/`; docs in `docs/`. Every command below runs verbatim from the repo root.

```bash
tools/fetch/fetch-found-sound.sh     # one-time: Internet Archive field recordings -> found/
tools/fetch/fetch-found-samples.sh   # one-time: SoundFont GM + breaks/one-shots/vox -> found/samples/
node tools/build/transcode-samples.js  # REQUIRED after a zone fetch: wav -> mp3 + re-bake SAMPLERS
./serve.sh                     # http://localhost:8777/  (serves index.html; needs http, not file://)
./verify.sh                    # orchestrator: forks 13 gate rows concurrently (list below)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aboard-io/stellate](https://github.com/aboard-io/stellate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
