---
trigger: always_on
description: This repo **is an agent skill** (`hve-video-director`) that runs on both **GitHub Copilot CLI** and **Claude Code**, not a typical application. The "source" is prompt content (markdown) plus Python helper scripts. There is no build system or lint config; pure-stdlib helper tests live under `test/`. The skill is consumed by future agent sessions that invoke `/hve-video-director <project-dir>` (a slash command on Claude Code; invoked by name/intent on Copilot CLI). The `SKILL.md` frontmatter follo
---

# Copilot Instructions — hve-video-director

## What this repo is

This repo **is an agent skill** (`hve-video-director`) that runs on both **GitHub Copilot CLI** and **Claude Code**, not a typical application. The "source" is prompt content (markdown) plus Python helper scripts. There is no build system or lint config; pure-stdlib helper tests live under `test/`. The skill is consumed by future agent sessions that invoke `/hve-video-director <project-dir>` (a slash command on Claude Code; invoked by name/intent on Copilot CLI). The `SKILL.md` frontmatter follows the Claude Code skill schema; Copilot CLI loads the skill from its `name`/`description` and harmlessly ignores the Claude-only fields (`allowed-tools`, `user-invocable`, `argument-hint`). See the **Runtime Compatibility** section in `SKILL.md` for how interaction blocks (`{"questions": […]}`), companion-skill loading (`Skill(<name>)`), and skill-home paths map across runtimes — preserve that mapping when editing.

The renderer is **HyperFrames** (HTML + GSAP, rendered via headless Chromium). React/Remotion are **not** used.

Keep two scopes distinct when editing:

- **This repo** — the skill definition (`SKILL.md`, `workflows/`, `templates/`, `patterns/`, `scripts/`, `design-systems/`). Edits here change behavior for *all* users.
- **Generated video projects** — created by the skill at runtime in `{project-dir}/`. They contain `project-plan.md`, `.hve/brief-state.json`, `context.md`, `storyboard.md`, `DESIGN.md`, `public/screenshots/`, `scenes/*.html`, `index.html` (root HyperFrames composition), `voiceover.mp3`, `out/final.mp4`. These do **not** live in this repo, and no reference build is committed while its replacement is being regenerated (see above).

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
- Phase 5 needs `index.html` (root composition) and passing `npx hyperframes lint` + `npx hyperframes check`
- Tutorial content mode prefers `public/clips/` but degrades to stills with a warning when clips are absent (warn-don't-block, spec §7.3); only missing captions is a hard check in tutorial mode

**External dependencies the skill calls out to:**

- `mcp__chrome-devtools__*` for app capture (Phase 2)
- The `hyperframes` companion agent skill for HTML/GSAP authoring rules (Phases 3 + 4) — distinct from the `hyperframes` npm CLI
- GSAP choreography reference lives in the `hyperframes-animation` skill (there is no standalone `gsap` companion skill)
- `npx hyperframes` CLI for `init`, `add` (pull catalog blocks, Phase 4), `lint`, `preview`, `check` (required final gate; `inspect`/`validate`/`layout` are deprecated aliases), `snapshot`, `render`, `doctor` (render-environment diagnostics, Phase 5), `transcribe` (preferred voiceover-timing verifier in Phase 5; falls back to standalone Whisper if unavailable), and `tts` (used in Phase 5 when the user explicitly confirms a local Kokoro voice)
- `mcp__chrome-devtools__screencast_*` + `resize_page` for Phase-2 web-clip capture (experimental, feature-detected — needs `--experimentalScreencast=true`; falls back to screenshots), and optional `asciinema`+`agg` for CLI clip recording (otherwise the authored-terminal path)
- `mcp__chrome-devtools__list_pages` + `select_page` for the explicit authenticated-session path. The user must first connect the MCP to running Chrome with Chrome 144+ `--autoConnect` (preferred) or the dedicated-profile `--browser-url` fallback; attached capture never navigates and follows `patterns/authenticated-browser-capture.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nebrass/hve-video-director](https://github.com/nebrass/hve-video-director) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
