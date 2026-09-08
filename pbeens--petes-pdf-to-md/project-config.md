---
trigger: always_on
description: Project-level operating instructions for Codex and other contributors working in this repository.
---

# AGENTS.md

Project-level operating instructions for Codex and other contributors working in this repository.

## Project Overview

`Pete's PDF to MD` is an Electron desktop app plus a scriptable conversion pipeline that turns PDF documents into Markdown better suited for AI and LLM workflows.

The current codebase focuses on:

- extracting a reliable heading outline from a PDF
- using that outline to split the PDF content into manageable Markdown files
- supporting both command-line and desktop GUI workflows
- producing output that is easy to inspect, review, and reuse

This project is not a generic PDF renderer. Its main job is to convert structured PDFs into high-quality Markdown organized by document headings.

## Main Technologies

- Node.js 20+ for the app shell, CLI launcher, and build scripts
- Electron for the desktop GUI
- Python 3.10+ for the PDF extraction engine
- PyMuPDF (`fitz`) for reading PDF structure and text
- `electron-builder` for packaged app builds

## Repository Layout

- `README.md`: end-user setup, build, and troubleshooting guide
- `CHANGELOG.md`: release history
- `package.json`: npm scripts, app version, Electron Builder config
- `app/`: renderer-side GUI files
- `electron/`: Electron main process and preload bridge
- `scripts/phase1.js`: Node launcher for the conversion pipeline
- `scripts/extract_outline.py`: Python extraction and Markdown segmentation logic
- `scripts/build-mac.sh`: macOS packaging helper
- `scripts/release-prep.sh`: release-readiness checks and optional macOS build trigger
- `docs/implementation-plan.md`: planning notes and implementation direction
- `test-data/pdfs/`: sample PDFs for local testing
- `.agents/skills/`: project-local skills

## How The App Works

There are two main entry points:

- CLI flow: `npm run phase1 -- --input "path/to/file.pdf"`
- GUI flow: `npm run gui`

Both flows ultimately run the same conversion pipeline:

1. `scripts/phase1.js` validates arguments, finds a usable Python interpreter, and checks that PyMuPDF is installed.
2. `scripts/phase1.js` invokes `scripts/extract_outline.py`.
3. `scripts/extract_outline.py` reads the PDF, extracts or infers headings, normalizes the outline, and writes Markdown outputs plus metadata files.
4. The Electron app reads those output files back from disk and displays the outline and section content.

## Core Application Files

### Electron

- `electron/main.cjs`: app bootstrap, window creation, IPC handlers, conversion subprocess spawning, output loading, shell integrations
- `electron/preload.cjs`: safe API bridge exposed to the renderer

### Renderer

- `app/index.html`: desktop UI markup
- `app/styles.css`: desktop UI styling
- `app/renderer.js`: renderer state, event handling, outline browsing, section preview, output-mode selection

### Conversion Pipeline

- `scripts/phase1.js`: CLI wrapper and Python/PyMuPDF environment resolution
- `scripts/extract_outline.py`: primary extraction engine and output writer

## Output Model

For an input PDF named `example.pdf`, output is written under:

- `<output-root>/example/`

Common generated files:

- `outline.json`: machine-readable outline and section metadata
- `outline.md`: human-readable outline summary
- `segments.json`: split planning / segment metadata

Depending on conversion mode:

- `single`: one merged Markdown file in the output root folder for that PDF
- `sections`: one Markdown file per heading in `Sections/`
- `major`: one Markdown file per major heading in `By Major Heading/`

In the packaged GUI, the default output root is:

- `~/Documents/Pete's PDF to MD Output`

## Development Workflows

### Install

Required local dependencies:

- Node.js 20+
- Python 3.10+
- PyMuPDF installed for the same Python interpreter the app will use

Typical setup:

- `npm install`
- `python3 -m pip install pymupdf`

### Run The GUI

- `npm run gui`

### Run The CLI Pipeline

- `npm run phase1 -- --input "test-data/pdfs/<file>.pdf"`

### Build

- macOS DMG: `npm run build:mac`
- Windows installer: `npm run build:win`

### Release Prep

- `npm run release:prep -- --version <x.y.z>`
- `npm run release:prep -- --version <x.y.z> --build`

`scripts/release-prep.sh` checks:

- `package.json` version matches the requested version
- `CHANGELOG.md` has a matching release section
- `README.md` references the release version
- `README.md` includes the GitHub Releases URL

## Local Skills Convention

This repository keeps project skills under:

- `.agents/skills/<skill-name>/SKILL.md`

When a user request clearly matches a local skill in `.agents/skills/`, open and follow that skill.

## Project Skill

### Available skills

- `petes-pdf-release`: prepare and publish a macOS GitHub release, including version checks, changelog/README release updates, and DMG build flow. File: `.agents/skills/petes-pdf-release/SKILL.md`

Use this skill when the task is about release preparation, DMG packaging, or release publication.

## Execution Rules

1. Prefer deterministic scripts over ad-hoc shell commands.
2. Keep scripts idempotent where possible.
3. When adding a new workflow, expose it through `scripts/` first.
4. Prefer extending the existing CLI or packaging scripts over embedding one-off behavior inside documentation.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pbeens/Petes-PDF-to-MD](https://github.com/pbeens/Petes-PDF-to-MD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
