---
trigger: always_on
description: Eidoverse is a collection of useful tools for **creating and rendering
---

# Eidoverse — Agent Guide

Eidoverse is a collection of useful tools for **creating and rendering
three.js videos in Deno, at real-time speeds, with absolutely minimal CPU
usage**. Everything renders on the GPU — WebGPU + NodeMaterial/TSL
throughout, no per-frame CPU loops, no baking — and the toolkit layers
simulation (fluids, water, cloth, particles), procedural builders
(creatures, robots, terrain, materials), a character controller,
audio generation, and post effects on top of that base. You are working in
this repo alongside a human collaborator; what to make comes from the
conversation.

Read this file completely before doing anything else. It is the single
agent-facing contract for the whole toolkit: the API, the production rules,
and the hard-won anti-patterns live here.

The goal for any video is a **finished produced short** — not a render
smoke test, not an isolated 3D clip. A complete piece of media: 3D scene +
(optionally) character + audio + motion graphics + a story arc that fills
the runtime.

**Rendering:** `python eido.py render <scene.json>` — native Deno + your
GPU, no containers. Iterate with single-frame probes (`--probe`) before
committing to full renders, and don't run two sustained renders
concurrently.

**Audio capabilities:** `generate_song.py` / `generate_sfx.py` need a
reachable ComfyUI backend — confirm with `python generate_song.py --probe`
(exits 0/1 in seconds). No ComfyUI → build the mix from edge-tts narration
(+ ffmpeg-synthesized ambience) or user-supplied audio. Never fake a tool
invocation; degrade honestly.

**Scratch space:** all your work goes in `work/<short_id>/` —
scene files, fetched assets, audio, intermediates, and the final mp4.
The engine files under `eidoverse/` are the canonical library every scene
shares — edit copies in `work/`, and change the engine itself only as a
deliberate, discussed decision.

## Duration — decide from the format, then fill it

Pick the runtime from the format before you build, and make the whole
piece earn it:

- **Landscape:** 45–90 seconds.
- **Portrait:** 60–120 seconds.
- **Music video:** the full length of the generated song.

Set `scene.json` `duration` to the runtime you chose, and build a 4–6
phase arc — intro, build, climax, resolve — with the camera moving
through it. The piece evolves across the whole runtime rather than holding
one shot.

Spread the narration evenly across the entire timeline, with a closing
line near the end, so the back half carries voice rather than bare music.
Set `duration` to at least the length of your mixed audio so the full
narration plays; the merge step keeps the audio intact.

If time is short, ship a simpler piece that still fills the format range —
one strong scene that runs the full minute beats an ambitious fragment.

## Mandatory production rules

These are non-negotiable — they are the difference between a finished
piece and a render test.

1. **Audio is required, fills the entire runtime, in correct mix balance.**
   - Generated music or instrumental bed (`generate_song.py`, when ComfyUI is available)
   - Plus TTS narration (`edge-tts` → `cyborg_voice.py` / `cyborg_stutter.py`) OR procedural SFX
   - **Voice 6-9 dB ABOVE the music bed** — never below
   - Audio runs from t=0 to end with no silent stretches; never front-loads narration and trails off into bare music

2. **Asset sourcing comes before procedural geometry.** Before building
   anything from primitives, in this order:
   - `python3 fetch_model.py "search terms"` — searches local custom models + Poly Haven + Smithsonian + NASA + NIH 3D **all at once, in parallel**, ranks every candidate across all sources, and delivers the best one (printing the runners-up from every source). **READ the preview before placing the mesh** — verify orientation + scale by using visible features plus the colored axis labels (+X red, +Y green, +Z blue).
     - **ALWAYS pass `--theme "<the brief's mood/setting>"`** so the pick fits your video, e.g. `fetch_model.py "car" --theme "cyberpunk neon dystopia"` or `fetch_model.py "vase" --theme "ancient cracked archaeological relic"`. Theme fit is **semantic** — an embedding model scores how well each candidate matches your setting by meaning, not keywords, so phrase the theme naturally (paraphrases, mood words, eras all work). The theme RE-RANKS the relevance-matched candidates: a damaged car floats up for a dystopia and sinks for a vintage showroom. It never promotes an off-query item (a clock won't win "chair") — it only reorders genuinely-relevant ones. *(Theme ranking uses `EIDOVERSE_EMBED_URL`/`EIDOVERSE_EMBED_MODEL`/`EIDOVERSE_EMBED_KEY` — any OpenAI-compatible `/v1/embeddings` endpoint, defaulting to Jina's free tier via `JINA_AI_KEY`; with no key it degrades to relevance-only, never errors.)*

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SkyeShark/eidoverse-video](https://github.com/SkyeShark/eidoverse-video) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
