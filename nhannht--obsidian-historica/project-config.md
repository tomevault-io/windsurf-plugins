---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Obsidian Historica is an Obsidian.md plugin that generates timelines from note content using NLP date extraction. It parses markdown text, extracts sentences with dates via chrono-node, and renders interactive timeline visualizations. Desktop-only. English-first, with auto-detected multilingual parsing (EN, DE, FR, JA, ZH, NL, VI).

## Task Tracking (YouTrack) - MANDATORY

All tracked work for this repo lives in YouTrack project **OBH** (`obsidian-historica`). This is the home; do not use any other tracker.

- Create an OBH issue for any work that outlives the session (feature, bug, multi-phase build) BEFORE starting it. Update its State as work progresses: `Open` → `In Progress` → `Fixed` (→ `Verified` once confirmed).
- Log work items (spent time) on the issue when a chunk of work completes (`log_work`).
- OBH uses the default scheme: Type {Bug, Feature, Task, Epic, …}, State {Submitted, Open, In Progress, …, Fixed, Won't fix, Verified}. Always call `get_issue_fields_schema` for OBH before `create_issue` / `update_issue`.
- The built-in session task list (`TaskCreate`/`TaskUpdate`) is for within-session steps only - it is NOT the backlog. Anything that must persist goes in OBH.

## Build & Dev Commands

```bash
bun install               # Install dependencies
bun run dev               # Dev mode (concurrent CSS watcher + esbuild watcher)
bun run build             # Production build (minified CSS + tsc check + esbuild)
bun test                  # Run all tests (Bun native runner, ~300ms)
bun test __tests__/ChronoParser.test.ts          # English date parsing only
bun test __tests__/ChronoParserDE.test.ts        # German (WikiWarsDE corpus)
bun test __tests__/ChronoParserFR.test.ts        # French (FTiB corpus)
bun test __tests__/ChronoParserJA.test.ts        # Japanese (hand-curated)
bun test __tests__/ChronoParserJAWiki.test.ts    # Japanese (Wikipedia coverage)
bun test __tests__/ChronoParserZH.test.ts        # Chinese (hand-curated + known gaps)
bun test __tests__/ChronoParserZHWiki.test.ts    # Chinese (Wikipedia coverage)
bun test __tests__/ChronoParserNL.test.ts        # Dutch (WikiWars-NL curated)
bun test __tests__/HmdParser.test.ts             # HMD storage format parse/serialize round-trip
bun run doc:code          # Generate TypeDoc documentation
```

The build produces three artifacts: `main.js`, `styles.css`, and `manifest.json`.

## Test Corpora

Multilingual NLP tests use real-world annotated corpora and Wikipedia articles. See `docs/test-datasets.md` for acquisition details, baselines, and known gaps per language.

## QA Testing (Obsidian CLI)

The test vault is at `historica-test-vault/`. QA uses the `obsidian` CLI which communicates with a running Obsidian instance over a local socket. Obsidian must be running (headless via Xvfb on servers).

### Setup

```bash
# Start Obsidian headless (if not running)
Xvfb :99 -screen 0 1920x1080x24 &
DISPLAY=:99 /opt/Obsidian/obsidian --no-sandbox &
# Wait ~5 seconds for startup
```

### Dev Workflow

`main.js` and `styles.css` are **symlinked** into the vault plugin dir — no copying needed.

```bash
# One-time setup (already done — skip if symlinks exist)
ln -s /home/larvartar/nhannht-projects/obsidian-historica/main.js historica-test-vault/.obsidian/plugins/historica/main.js
ln -s /home/larvartar/nhannht-projects/obsidian-historica/styles.css historica-test-vault/.obsidian/plugins/historica/styles.css

# Start dev watcher (auto-rebuilds on save)
tmux new-window -n 'historica-dev' 'bun run dev'

# QA: reload plugin in Obsidian (after any rebuild)
bun run qa
```

### Production Build

```bash
bun run build   # minified CSS + tsc check + esbuild
```

### Visual Verification (verify via DOM, NOT screenshots)

Do NOT take screenshots to check rendering. Verify via the DOM and computed styles -
deterministic, non-intrusive, and a better test than an image for CSS/token changes.

```bash
# Query DOM to verify the timeline rendered
obsidian dev:dom selector=".historica-timeline" text
obsidian dev:dom selector=".historica-timeline" all    # all matches

# Assert resolved styling / Int tokens via computed styles
obsidian eval code="getComputedStyle(document.querySelector('.historica-timeline')).backgroundColor"

# Inspect CSS
obsidian dev:css selector=".historica-timeline" prop=display
```

Screenshots: `obsidian dev:screenshot` / `dev:cdp Page.captureScreenshot` HANG whenever the
Obsidian window is occluded (a real X11 limitation - the compositor stops producing frames for
hidden windows). Do NOT work around it by raising the window (`Page.bringToFront`, steals focus)
or by grabbing the X display (`import`/`scrot`/`ffmpeg x11grab`, captures the user's live
screen). If a screenshot is genuinely required, ask the user to take it. See
`~/.claude/rules/obsidian-qa.md`.

### Error Checking

```bash
obsidian dev:console level=error    # JS console errors
obsidian dev:errors                 # captured runtime errors
obsidian dev:console                # all console output
obsidian dev:console clear          # clear buffer
```

### Test Note Management

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nhannht/obsidian-historica](https://github.com/nhannht/obsidian-historica) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
