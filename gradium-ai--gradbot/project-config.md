---
trigger: always_on
description: Screen guide + vision LLM + Gradium voice: **the app IS the guide.** The parsed
---

# LEGO Voice Assist — agent guide

Screen guide + vision LLM + Gradium voice: **the app IS the guide.** The parsed
manual's pages render on screen (the physical booklet leaves the loop), so the
step you're looking at is deterministically the current step — no camera reads a
booklet. A webcam watches only your messy PILE; the voice companion finds this
step's pieces in it and says where they are out loud — from *your* point of view
("close to you, on your right"). The official LEGO app tells you what to build;
this tells you **where the piece is in the jungle**, hands-free. The whole thing
is a showcase for Gradium's voice stack (STT ↔ conversational LLM ↔ TTS via
gradbot) in a hands-busy, eyes-busy setting.

<!-- gradium:generated -->
## Repo map

- `main.py` — the whole backend (FastAPI). Vision calls (`_vision_json`, Responses
  API), prompts, `/api/find` (pile search — backend smoke path), `/api/manual`
  (PDF parse + cache + per-page raster persist), `/api/manual/{id}/page/{n}`
  (serves a rasterized page — the on-screen guide's image), egocentric direction
  composition (`_egocentric_phrase`/`_build_speech`), `/ws/chat` (gradbot voice
  session: streaming Gradium STT/TTS + conversational LLM + tools `next_step` /
  `find_step` / `find_one_piece` / `check_piece` / `switch_language`),
  `/api/frames/{sid}` (pile-crop mailbox; `warm` prefetches the current step,
  `step: N` sets `state.step_idx` and prefetches step N — the on-screen prev/next
  control). `_prefetch(state, idx)` is the latency play: one pile search for the
  known step, then a look-ahead find for step N+1 so the next "next" is a cache
  hit.
- `static/index.html` — the whole frontend, single file, no build step: three
  panes — Companion (mic + last agent line + language buttons + settings), Guide
  (parsed manual pages rendered on screen, prev/next), Pile (webcam = the whole
  frame; 3×3 grid overlay + pinpoint highlights, driven by `setInterval` not rAF).
  gradbot call UI (SyncedAudioPlayer), capture round-trip (pile crop + full-frame
  `scene` for check_piece), manual upload/restore.
- `data/` — parsed-manual cache: `<id>.json` (parsed steps) + `<id>/page-NNN.jpg`
  (page renders), gitignored. `e2etestmanual.json` is a synthetic 3-step fixture;
  its `e2etestmanual/page-00N.jpg` renders back the on-screen guide in the harness.
- `demo/` — local screen recordings (gitignored, 65MB+; never commit).
- `.claude/launch.json` — dev-server launch config (autoPort; falls back when
  8410 is taken).
- `.env` (gitignored) / `.env.example` — `GRADIUM_API_KEY`, `OPENAI_API_KEY`,
  optional `OPENAI_MODEL`, `OPENAI_FAST_MODEL`, `OPENAI_REASONING_EFFORT`,
  `OPENAI_FIND_EFFORT`, `GRADIUM_VOICE_ID` (+ `GRADIUM_VOICE_ID_FR`/`_ES`/
  `_DE`/`_PT` per-language overrides), `GRADIUM_API_URL`, `GRADBOT_FLUSH_S`
  (STT flush window, default 0.3 — the end-of-speech→answer lever), and for
  the gradbot session `LLM_MODEL`/`LLM_BASE_URL`/`LLM_EXTRA_CONFIG` (defaults
  are derived from the OpenAI vars).

## Commands

```bash
uv sync                                         # install deps
uv run uvicorn main:app --reload --port 8410    # run (any port works)
uv run python -c "import main"                   # fast backend sanity check
```

No test suite and no linter are configured. Verification is manual — see
"How we verify" below.
<!-- /gradium:generated -->

## Architecture pointers

One FastAPI process serves the static page and two upstream stacks: OpenAI
vision (Responses API) for all pile lookups, and a gradbot session per call
(Rust multiplexer streaming Gradium STT ↔ conversational LLM ↔ Gradium TTS,
with turn-taking and barge-in). The conversational LLM never sees images — it
calls tools (`next_step`, `find_step`, `find_one_piece`, `check_piece`,
`switch_language`); tool handlers fetch fresh pile crops via a capture
round-trip (websocket `capture` event → browser POSTs crops to
`/api/frames/{sid}`), run the vision pipeline, and return a pre-composed `say`
sentence the agent relays verbatim. All keys stay server-side. The core design
insight: never identify *every* piece in the pile — a parsed manual is required,
and it says which few pieces the current step needs; the vision model only
localizes those on a labeled 3×3 grid burned into the pile crop — each copy as
`{cell, box}`, where the tight normalized bounding box drives a pinpoint
on-screen highlight (measured 0.1–0.2 % center error at high image detail) and
the cell stays authoritative for speech and as the visual fallback: a box only
survives `_piece_cells` if its centroid lands in the reported cell, which
provably drops exactly the hallucinated boxes. Deterministic code, never a
model, converts grid cells into builder-relative directions.

**The step is on the screen, not in the camera.** `_parse_manual` rasterizes
every PDF page and persists it (`data/<id>/page-NNN.jpg`); the guide renders
those pages, so the current step is deterministic (`state.step_idx`) — no spread
reads, no page-flip detector, no step matching, no camera-only mode. Voice
"next" and the on-screen prev/next control converge on the SAME server-side
index: prev/next POSTs `{step: N}` to the frames mailbox (→ `step_idx = N` +
prefetch); voice `next_step` advances the index and pushes the new page +

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gradium-ai/gradbot](https://github.com/gradium-ai/gradbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
