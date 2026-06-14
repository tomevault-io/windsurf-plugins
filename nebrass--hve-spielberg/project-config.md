---
trigger: always_on
description: This repo **is a Claude Code skill** (`hve-spielberg`), not a typical application. The "source" is prompt content (markdown) plus two Python helper scripts. There is no build system, no test suite, no lint config — the skill is consumed by future Claude Code sessions that invoke `/hve-spielberg <project-dir>`.
---

# Copilot Instructions — hve-spielberg

## What this repo is

This repo **is a Claude Code skill** (`hve-spielberg`), not a typical application. The "source" is prompt content (markdown) plus two Python helper scripts. There is no build system, no test suite, no lint config — the skill is consumed by future Claude Code sessions that invoke `/hve-spielberg <project-dir>`.

The renderer is **HyperFrames** (HTML + GSAP, rendered via headless Chromium). React/Remotion are **not** used.

Keep two scopes distinct when editing:

- **This repo** — the skill definition (`SKILL.md`, `workflows/`, `templates/`, `patterns/`, `scripts/`, `design-systems/`). Edits here change behavior for *all* users.
- **Generated video projects** — created by the skill at runtime in `{project-dir}/`. They contain `project-plan.md`, `context.md`, `storyboard.md`, `DESIGN.md`, `public/screenshots/`, `scenes/*.html`, `index.html` (root HyperFrames composition), `voiceover.mp3`, `out/final.mp4`. These do **not** live in this repo (except the canonical reference build under `example/`).

## Architecture

`SKILL.md` is the orchestrator prompt. It loads first, decides the entry mode (`new` / `continue` / `jump`), and dispatches to one of six phase workflows. Each phase has a user-approval checkpoint before advancing.

```
SKILL.md (orchestrator)
  ├─ workflows/phase-0-discovery.md     → produces context.md
  ├─ workflows/phase-1-storytelling.md  → produces storyboard.md
  ├─ workflows/phase-2-capture.md       → produces public/screenshots/ (via Chrome DevTools MCP)
  ├─ workflows/phase-3-design.md        → produces DESIGN.md + scenes/*.html (via hyperframes skill)
  ├─ workflows/phase-4-production.md    → produces root index.html composition (via hyperframes skill)
  └─ workflows/phase-5-audio.md         → produces voiceover.mp3 + background-music.mp3 + out/final.mp4
                                          (npx hyperframes render)
```

**Phase prerequisites are enforced in `jump` mode** — see `SKILL.md`. When editing workflows, preserve the file-presence contract:

- Phase 1 needs `context.md`
- Phase 2 needs `context.md` + `storyboard.md`
- Phase 3 needs capture artifacts (`public/screenshots/` and/or `public/clips/`)
- Phase 4 needs context + storyboard + `DESIGN.md` + `scenes/*.html`
- Phase 5 needs `index.html` (root composition) and passing `npx hyperframes lint|inspect|validate`
- Tutorial content mode prefers `public/clips/` but degrades to stills with a warning when clips are absent (warn-don't-block, spec §7.3); only missing captions is a hard check in tutorial mode

**External dependencies the skill calls out to:**

- `mcp__chrome-devtools__*` for app capture (Phase 2)
- The `hyperframes` Claude Code skill for HTML/GSAP authoring rules (Phases 3 + 4) — distinct from the `hyperframes` npm CLI
- The `gsap` skill (optional companion) for choreography reference
- `npx hyperframes` CLI for `init`, `add` (pull catalog blocks, Phase 4), `lint`, `preview`, `inspect`, `validate`, `render`, `doctor` (render-environment diagnostics, Phase 5), `transcribe` (preferred voiceover-timing verifier in Phase 5; falls back to standalone Whisper if unavailable), and `tts` (used in Phase 5 as the no-API-key fallback when `ELEVENLABS_API_KEY` is unset)
- `mcp__chrome-devtools__screencast_*` + `resize_page` for Phase-2 web-clip capture (experimental, feature-detected — needs `--experimentalScreencast=true`; falls back to screenshots), and optional `asciinema`+`agg` for CLI clip recording (otherwise the authored-terminal path)
- `scripts/generate_voiceover.py` → ElevenLabs API + optional Whisper transcription (Phase 5)
- `scripts/search_music.py` → Freesound API for CC music (Phase 5)

`templates/` files are copied into generated projects. `patterns/` files are referenced for visual techniques. `patterns/INDEX.md` is the wayfinding map between local patterns and the deeper `hyperframes` skill references — read it before adding new pattern files. `patterns/metallic-swoosh.md` documents *why* clipPath transitions are banned (black-sliver artifacts).

`design-systems/<slug>/DESIGN.md` is the brand spec consumed by Phase 3 Path A — MIT-licensed, video-focused, authored by this skill. The canonical research source for new contributions is [VoltAgent/awesome-design-md](https://github.com/VoltAgent/awesome-design-md) (MIT, 73 brands, has a `npx getdesign add <slug>` CLI). The skill is **video-only** — it does not produce, render, or analyse web/UI artifacts.

## Working with the skill scripts

The Python scripts run inside generated video projects, not from this repo. They self-install `requests` via pip on first run.

```bash
# Voiceover generation (from inside a generated project)
ELEVENLABS_API_KEY=... python3 scripts/generate_voiceover.py

# Music search (from inside a generated project)
FREESOUND_API_KEY=... python3 scripts/search_music.py
```

Both `ELEVENLABS_API_KEY` and `ELEVEN_LABS_API_KEY` are accepted (back-compat).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nebrass/hve-spielberg](https://github.com/nebrass/hve-spielberg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
