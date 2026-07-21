---
trigger: always_on
description: This file is the canonical guidance for AI coding agents (Claude Code, Codex,
---

# AGENTS.md

This file is the canonical guidance for AI coding agents (Claude Code, Codex,
etc.) working in this repository. `CLAUDE.md` imports this file and adds
Claude Code-specific notes — shared guidance belongs here, not there.

Agent skills live in `.agents/skills/` (one directory per skill with a
`SKILL.md`). `.claude/skills` is a symlink to that directory so Claude Code
picks them up — edit skills only in `.agents/skills/`.

## Mission & Map

**Goal: finish Virtua Grotesk and publish it to Google Fonts.** This repo is a
small AI-driven type studio — an agent (you) orchestrates the tools below to
draw, space, QA, and ship the font.

The phases and the tools for each:

- **Draw / fix glyphs** — `img2bez` traces from reference images (see "Adding
  Glyphs from Images"); Runebender (`make runebender`) is the visual review +
  live-edit surface. Skills: `/draw-outline`, `/edit-glyph`, `/compare-reference`,
  `/glyph-ai-harness`.
- **Space & kern** — per-glyph sidebearings in the UFOs; `/kerning`.
- **Build & QA** — `make build`, `make test` (the Fontspector `googlefonts`
  gate), `make proof` / `make specimen`, `make preflight`; skill `/font-qa`.
- **Package & submit** — `/google-fonts-packaging` (produces `METADATA.pb` + the
  `ofl/virtuagrotesk` layout), `/google-fonts-onboarding`, `/google-fonts-qa`.

**The finish line and current priorities live in
[`documentation/google-fonts-readiness.md`](documentation/google-fonts-readiness.md)
— read it first.** In short: the build, Latin, kerning, and the Arabic OpenType
shaping are done; what remains, in order, is (1) the **Arabic outline cleanup
pass** (top priority — the bulk of the work left), (2) Latin language-coverage
anchors, (3) `METADATA.pb` + packaging, (4) the `google/fonts` PR. Progress is
measured by the excludes in `scripts/check_gf_fonts.sh`: each one removed is a
step toward done, and **zero excludes = ready to submit.**

**Guardrails:** keep both masters structurally identical (master compatibility),
and never re-add a QA exclude to force a green `make test` — the excludes are the
to-do list, not a setting.

## AI Glyph-Completion Harness

This repo is the demo font for a unified AI type-production pipeline
(img2bez, img2ufo, designbot, an image-generation model, and this repo's
harness). Three documents define it:

- **`DESIGN.md` (repo root)** — the design contract: power-of-two grid,
  16-unit chamfers, metrics, curve/spacing rules. Every drawn or traced glyph
  is judged against it.
- **`harness/RUNBOOK-codex.md`** — the operating procedure for adding or
  regenerating **one glyph** from a reference image (trace with
  `img2bez masters` on a scratch copy, adjust per `DESIGN.md`, port into
  sources in repo style, mark **blue**, verify). If you were asked to add,
  regenerate, or trace a glyph, follow this runbook.
- **`plans/ai-font-completion-harness.md`** — the full system plan, research,
  mark-color protocol, and phase checklist.
- **`documentation/design-pass-worklog.md`** — the running glyph-by-glyph
  review of sources against the published blog contract (measurements,
  decisions and their reasons, OPEN items). Read it before touching A–Z/a–z;
  append an entry whenever a review or design decision happens. OPEN items
  are Eli's to resolve — agents measure and propose, never settle them by
  editing sources.

Mark colors in the UFOs are the human's control channel: green = done
(never touch), yellow/orange = needs polish, red = broken/regenerate,
blue = AI output awaiting human grading, no color = ignore.

## Project Overview

Virtua Grotesk is an open-source variable font (OFL v1.1 licensed) with a Weight axis (wght 400–700). The sources are UFO files and the Google Fonts-ready build path uses `gftools builder sources/config.yaml`.

## Quick Start

```bash
/build-font             # Build all fonts (variable + static)
/proof                  # Generate PDF proof document
make specimen           # Generate landscape print spacing specimen
make reports            # Regenerate source/build metadata reports
make preflight          # Build, proof, specimen, reports, then check artifacts
make test               # Build, then run Fontspector googlefonts profile
/edit-glyph A           # Inspect/edit a glyph
make runebender         # Open the font in the Runebender web editor —
                        # edits to sources/ on disk live-reload in the
                        # browser; the user's Cmd+S saves back to disk
/kerning list           # View current kerning pairs
/compare-reference img  # Compare font to a reference image
```

## Font Metrics

| Metric | Value |
|--------|-------|
| Units per Em | 1024 |
| Ascender | 832 |
| Cap Height | 768 |
| x-Height | 576 |
| Descender | -256 |
| Grid Size | 2 (prefer even coordinates) |

## Build Commands

**Prerequisites:** Python virtual environment at `.venv/` with `pip install -r requirements.txt`.

```bash
make setup      # Create .venv and install requirements
make build      # Build variable and static TTFs into fonts/
make proof      # Build documentation/proofs/proof.pdf
make specimen   # Build documentation/proofs/print-spacing-specimen.pdf
make runebender # Open sources/VirtuaGrotesk.designspace in Runebender web

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eliheuer/virtua-grotesk](https://github.com/eliheuer/virtua-grotesk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
