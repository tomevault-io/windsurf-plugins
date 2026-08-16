---
trigger: always_on
description: Open source Screen Studio alternative. Two parts: a native macOS recorder
---

# CLAUDE.md — Retake

Open source Screen Studio alternative. Two parts: a native macOS recorder
(Swift CLI) and a one-file browser editor (canvas compositor + WebCodecs).
Named 2026-07-30 ("Record once. Retake forever."). Binary is `retake`,
bundles are `.take`; legacy `.osrec` bundles must always keep opening (the
loader keys off bundle contents, never the extension — do not add one).

## Layout

- `Recorder/` — Swift Package CLI (`retake`). ScreenCaptureKit,
  cursor logged as DATA (never baked into pixels), system audio, --window/
  --area/--keys. Builds only on macOS: `swift build -c release`, then
  `sudo cp .build/release/retake /usr/local/bin/ && hash -r`.
- `retake-editor.html` — the SHIPPED editor (single self-contained file).
  Built from source: `Editor/retake-editor.html` (markup/CSS) +
  `Editor/app.js` (logic) + inlined mp4-muxer.js/gifenc.js via
  `cd Editor && node build.js` → `Editor/dist-retake-editor.html`,
  then copy dist over the root file to ship.
  ⚠️ ALWAYS run build.js before testing — tests load the dist file.
  Never hand-edit the root file or the dist; edit the sources.
- `test/` — Playwright suite. `*.test.mjs` files are gates (`npm test` runs
  all twenty); `shot-*.mjs` are screenshot helpers, not gates. The runner is
  a hardcoded list in `test/package.json` — a new `.test.mjs` file does NOT
  run until it is added there, so add it in the same commit or it is inert. Paths resolve
  through `test/paths.mjs`, so the suite runs on any checkout. Setup once:
  `npm install` and `npx playwright install chromium`. Run: `npm test` from
  `test/`. Fixture: `test/fixture.take` (VP9/Opus because Playwright's
  Chromium lacks H.264/AAC — real Chrome uses H.264/AAC; the editor has
  fallbacks). Regenerate with `./make-fixture.sh`. Run ALL gates after any
  editor change.

## This repo holds CODE. Nothing else.

**No state files, no session notes, no roadmap, no private observations, and no
absolute paths into the author's home directory.** This repo is public and
permanent; project thinking lives outside this repo. This file (the contract) and `README.md`
(the shipped artifact, rule 12) are the only prose that belongs in it.

That applies to **commit messages and code comments too**, which are just as
public as the files. Write them for a stranger reading the repo cold: no
internal role words, no references to documents that only exist outside this
repo. A comment pointing at a note the reader cannot open is worse than no
comment.

Before writing any note-shaped file here, stop. It belongs outside the repo.

**Before publishing anything, enumerate — do not search.** Grep only finds
words you already suspect, and the working tree looking clean says nothing
about history. List every path in `git rev-list --all --objects`, read every
commit message body in full, open every binary and look at it. And never
report something as removed from a remote without fetching its public URL and
seeing it fail: a fresh clone looks clean whether or not the host still serves
the old objects, so a clone is not a test.

## Hard rules

1. **Edits are data; the source video is sacred.** Cuts/zooms/crop are state
   mapped at render time (`out2src`/`src2out`, `cropRect()`, camera blending).
   Never mutate or re-encode source media in edit operations.
2. **Privacy default**: key *timings* only; actual keys ONLY behind `--keys`.
   Nothing ever leaves the machine. No analytics, no network calls.
3. **Single-file editor**: no external deps, fonts, or CDNs. New libs must be
   inlined via build.js.
4. **Test contract**: element ids (exportBtn, exportFmt, exportQ, exportRes,
   exportFps, sizeEst, dirInput, timeline, splitBtn, startBtn, helpBtn, tip, tlZoom,
   cropW/cropH, cursorStyle data-v
   buttons, cropAlignH/cropAlignV data-a buttons, tlHelp, saveLook, lookHistory
   (+ .look-row buttons), keysHint, the sidebar section wrappers
   secFrame…secLook…) and global functions (pause,
   seekTo, addZoomAt, splitAtPlayhead, deleteSelection, undo, cameraAt,
   outDuration, out2src, enterCropMode…) are load-bearing. Keep native <input>
   elements native (tests drive them with value + dispatchEvent('input')).
5. **Popover/overlays**: hide with opacity/pointer-events, NEVER display:none
   (Playwright actionability). backdrop-filter only while `.open` (perf).
6. **Design system (Aurora-hybrid)**: gradient = interactive, everything
   static stays graphite; elevation by luminance (4 surface tokens); one
   spring (300ms), one ease (150ms); nothing animates during playback/export.
   Two sanctioned exemptions to the gradient rule, decided 2026-07-30: the
   brand mark (`#logo i`) and the export progress fill. Don't "fix" them.
   Text tokens must clear 4.5:1 on the surface they land on (`--faint` is the
   floor at 4.96 on `--s1`); canvas text carries its own hardcoded hexes.
7. **Undo**: structuredClone snapshots via pushUndo() BEFORE mutations; one
   push per drag (pointerdown); baseline seeded after load (undoReady).
   Any new edit operation must call pushUndo(). Scope: undo covers timeline
   and crop edits (what editSnapshot holds) — style keys are outside it by
   design, so style-only operations (look apply/promote, sliders, toggles)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tonhaoln/retake](https://github.com/tonhaoln/retake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
