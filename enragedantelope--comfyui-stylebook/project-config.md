---
trigger: always_on
description: Visual styles for ComfyUI, every one with a rendered preview you can browse before you commit. Each ships a written description, a keyword list, a matching negative prompt, plus artists with descriptors. Zero dependencies, fully offline. Built on ComfyUI V3 API, category: `conditioning/stylebook`.
---

# AGENTS.md — comfyui-stylebook

Visual styles for ComfyUI, every one with a rendered preview you can browse before you commit. Each ships a written description, a keyword list, a matching negative prompt, plus artists with descriptors. Zero dependencies, fully offline. Built on ComfyUI V3 API, category: `conditioning/stylebook`.

**Deep references:**
- `ARCHITECTURE.md` (chain protocol, layout, design rationale — read before engine changes)
- `docs/custom-styles.md` (field reference for `user_styles.json`)

## Current state

_Last verified: 2026-10-02_

- **Status:** `main` is v0.16.1. v0.17.0 is built on branch `revision-0.17.0` and awaits the maintainer's testing; nothing merges to `main` until they say so. Published to ComfyUI Manager but unadvertised. `.github/workflows/publish_action.yml` fires on a `pyproject.toml` version change on `main` — a commit touching nothing the registry ships needs no bump. `.comfyignore` says what the registry package leaves out; its patterns are gitignore-style, so root-only ones carry a leading slash.
- **Works:** all five nodes (Style, Artist, Modifier, Blend, Sheet) over the `STYLEBOOK_CHAIN` protocol; a rendered preview tile for every style and every modifier on all six axes, packed into WebP sprite atlases; the two-line node-face readout plus Copy-resolved-prompt and Pin-this-pick context items; the right-click Auto-advance cycle toggle (advances once per queued item, so Batch count walks consecutive entries — see `ARCHITECTURE.md`); the "New in x.y.z" tab, new ribbon and A-Z/Newest sort driven by `data/versions.py`; optional `user_styles.json` validated and merged at load; the public gallery plus artist and modifier reference pages on GitHub Pages; the full CI gate including a jsdom frontend suite and a no-GPU preview `--check`. `python tests/validate_data.py` reports the true totals.
- **In progress:** 0.17.0 review. It fixes Sheet and Blend ignoring a style's `blocks`, the always-re-run `fingerprint_inputs` on every node (it made every downstream node, the sampler included, re-execute each queue), modifier Cycle order, and an untested `execute()` path; moves the examples to core nodes; and adds artists, styles and finish/lighting modifiers. Two modifiers were tried and dropped after A/B renders (Kaleidoscope rendered the toy, not the look).
- **Known gaps / next steps:** the Dragon Ball tile draws a house-style character whatever the wording (documented as the limit of the franchise rule); the Porcelain Figurine tile shows the material but a faceless figure; Copy, Pin and the Cycle pool size do not reach a node inside a subgraph (its execution id looks like `12:5` and the frontend matches on the plain node id); with no `fingerprint_inputs` a cached node sends no event, so its readout stays blank after a browser reload until an input changes. Settled non-gaps, worth not re-deriving: the `era` and `mood` axes are closed to new records and `color_grade` was audited with nothing to add (`ARCHITECTURE.md` says why); artist preview tiles are declined; there is no CONDITIONING-output node. Operational facts: `build_previews.py --build` exits non-zero when it cannot reach ComfyUI — trust it only when run in the foreground, since a backgrounded wrapper reports its own status; it refuses to render without an explicit `--model`; ComfyUI caches the Python data layer at startup, so new entries fail node validation until it restarts; the namesake detector cannot see an adjectival label ("Sirkian Melodrama"), so those are declared by hand; `--prune` clears removed styles but not a removed modifier's manifest entry, which is deleted by hand.
- **Deep docs:** `ARCHITECTURE.md` (design rationale, writing rules, caching and cycle mechanics), `docs/custom-styles.md` (`user_styles.json` fields).

## Architecture in 60 seconds

- **Chain protocol.** Every node takes an optional `style_chain` and emits one on a dedicated `STYLEBOOK_CHAIN` socket type (not STRING — prevents silent miswiring). Carries JSON: style + modifiers + artists + user_prompt metadata.
- **Five nodes.** Style (exclusive medium axis), Artist (additive, chainable), Modifier (one per axis), Blend (two styles at a ratio), Sheet (one subject, many styles as a list).
- **Gallery-first UX.** Each style ships a rendered preview image. Open gallery → look → click. Every artist has a written descriptor so the look lands even when the model doesn't know the name.
- **Three composition rules** (differ on purpose): style = exclusive replacement; modifiers = per-axis additive; artists = chainable additive.
- **Plain Python + plain JavaScript.** No OS-specific calls, paths via `pathlib`, nothing shells out. Runs the same on Windows/macOS/Linux. Test suite runs on a machine with no ComfyUI installed.
- **Generated JS data.** `js/stylebook_data.js` is generated — never edit by hand. `js/stylebook_gallery.js` is hand-written and the generator never touches it.

## Layout

| Directory / File | Purpose |
|------------------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EnragedAntelope/comfyui-stylebook](https://github.com/EnragedAntelope/comfyui-stylebook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
