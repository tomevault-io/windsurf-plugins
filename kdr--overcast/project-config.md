---
trigger: always_on
description: Guidance for Claude Code / pi / any agent working in this repo — the quick map +
---

# CLAUDE.md

Guidance for Claude Code / pi / any agent working in this repo — the quick map +
the invariants you must not break. `overcast commands --json` is the authoritative
verb surface; verify against it, not memory.

## What this repo is

**overcast** — a portable toolkit that gives an agent *senses* (video / audio /
image understanding) and *OSINT reach* (search / capture / monitor), organized
around an investigation **case**. Built **on top of
[pi](https://github.com/earendil-works/pi)** (the agent harness), with **tinycloud
/ Cloudglue** as the default perception backend.

It ships three ways from one source of truth (`src/registry/verbs.ts`): a **pi
package** (extension + skills + prompts + theme), a **standalone bun binary**, and
**agent skills** that drive the CLI from any harness.

## Stack (pinned)

- `@earendil-works/pi-ai`, `pi-agent-core`, `pi-tui`, `pi-coding-agent` —
  **exactly `0.82.1`**. Don't float these; treat upgrades as reviewed changes.
- `@cloudglue/cloudglue-js` — the default sense backend (via the tinycloud CLI,
  `exec`). Cloudglue is **also** a pickable *brain* LLM provider (anthropic-messages
  API) so it appears in `/model` — never forced. The tinycloud CLI is a runtime
  prerequisite (like ffmpeg), not an npm dep; the floor is tinycloud
  **≥ 0.3.12** (`watch.speech.v1`) — the watch envelope inlines VERBATIM speech
  (`segments[].speech`), so `watch`/`listen` transcripts are single-call
  (`listen --diarize` still rides the public `caption` verb, which is also the
  fallback for 0.3.10/0.3.11 envelopes that ship `segments: []`); `face` +
  `index` need ≥ 0.3.4, and image `see`/`extract` (the opt-in `see:tinycloud`
  provider) need ≥ 0.3.7.
- `ffmpeg` + `ffprobe` — a **system prerequisite** (on `PATH`, or via
  `OVERCAST_FFMPEG` / `OVERCAST_FFPROBE`); the internal media toolkit, NOT bundled.
- uv-managed visual/audio DB Python — optional for visual/audio DBs,
  `face:deepface-local`, and the `enhance:local-models` split ops:
  `scripts/visual-db-uv.sh --face` installs OpenCV/Numpy and DeepFace/TensorFlow;
  `--clip` adds OpenAI CLIP (open_clip + torch + pillow) for the `basic-clip`
  semantic DB; `--detect` adds the OWLv2 open-vocab detector (torch + transformers
  + scipy + pillow) that backs `see --detect` (set `DETECT_PY` to the venv);
  `--audio` adds scipy for the `audio-fp` Shazam-style fingerprint DB; `--clap`
  adds LAION CLAP (transformers + torch) for the `basic-clap` audio-embedding DB;
  `--voice` adds pyannote.audio (`enhance --ops separate` + the `voice-print`
  speaker-verification DB for `voice`), `--segment` adds
  transformers + SAM2/GroundingDINO (`enhance --ops segment`), `--enhance` adds both
  enhance stacks, `--all` installs everything. Override with `OC_VISUAL_DB_PY` /
  `OVERCAST_VISUAL_DB_PY`. Voice separation and `voice match --diarize` additionally
  need `HF_TOKEN` + accepted pyannote license (the windowed `voice` default is
  ungated).
- TypeScript / ESM / Node ≥22; `tsup` (dev build) + `bun build --compile` (binary).

## Invariants (do not violate)

1. **Don't fork pi.** Reuse pi's loop, TUI, sessions, base tools
   (`read/write/edit/bash/grep/find/ls`), and provider layer. overcast attaches as
   a pi **package/extension**; net-new code is the verbs + providers + record store.
2. **BYO LLM.** Never hardcode the brain provider. Keep the *brain provider*
   (pi-ai) and the *sense providers* (tinycloud / VLM / STT) separate everywhere.
   *One deliberate, opt-out bridge:* `see` defaults to the **brain LLM** for image
   description when it's image-capable (`src/providers/brain/vision.ts`) — it
   resolves whatever brain the profile/env already points at (BYO, never a
   hardcoded one) and is one switch away from the classic sense provider
   (`setup provider see builtin:hf` / `OVERCAST_SEE_BRAIN=off`). Don't extend this
   pattern to other verbs without the same "resolved-not-hardcoded + opt-out" bar.
3. **The record is loose.** Output contract = `{ id, verb, format (json|md|txt),
   payload, media?{ref,at}, meta?, error?, state? }` and nothing more. Map provider
   output to the record at the exec boundary; never reintroduce a rigid envelope.
   `state`/`error` are the only optional control fields; a missing `state` = `ready`.
4. **Case = a folder.** No bespoke case object — a case is a directory with a
   `.overcast/` store; pi's per-directory sessions are the case history. Switch
   cases by `cd` or `--case <dir>`.
5. **One verb spec → three surfaces.** Declare each verb once in
   `src/registry/verbs.ts`; the CLI subcommand, the pi AgentTool, and the skill doc
   are generated from it. `overcast commands --json` is the source of truth.
6. **Providers are pluggable.** Three classes share one machinery — **sense**
   (`watch/listen/see/face/cluster/image/audio/voice/similar/enhance/reconstruct/exif/verify/screenshot/chronolocate` —
   `cluster` shares the face provider, `chronolocate` is pure local math), **source**
   (`scan/capture/monitor`; youtube, tiktok, x, web, lens, yandeximg, dl, instagram, telegram,
   gdelttv, overpass, firms, dispatch, flights, webcam, facesearch, dork, shodan, browser,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kdr/overcast](https://github.com/kdr/overcast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
