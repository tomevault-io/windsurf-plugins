---
trigger: always_on
description: [Generated from codebase reconnaissance on 2025-12-18]
---

# A0 Launcher - AGENTS.md

[Generated from codebase reconnaissance on 2025-12-18]

## Quick Reference
- **Tech Stack**: JavaScript (CommonJS) | Electron 33.x (Electron Forge 7.6.x) | Node.js 20+ | GitHub Actions
- **Dependency highlights**: `electron@^33.2.0`, `@electron-forge/*@^7.6.0`, `electron-squirrel-startup@^1.0.1`
- **Dev Run**: `npm start` (GUI required)
- **Build (local)**: `npm run make` (or `npm run make:<os>`)
- **Fork E2E Content Repo Override**: `A0_LAUNCHER_GITHUB_REPO="owner/repo" npm start`
- **Local Dev Content (skip GitHub Releases)**:
  - `A0_LAUNCHER_USE_LOCAL_CONTENT=1 npm start` (use CWD if it contains `app/index.html` + `package.json`)
  - `A0_LAUNCHER_LOCAL_REPO="/abs/or/relative/path" npm start` (override repo dir; takes precedence)
- **Docs**: `README.md`, `.specify/memory/constitution.md`, `specs/`

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Core Commands](#core-commands)
3. [Project Structure](#project-structure)
4. [Development Patterns and Conventions](#development-patterns-and-conventions)
5. [Safety and Permissions](#safety-and-permissions)
6. [Code Examples (Good and Avoid)](#code-examples-good-and-avoid)
7. [Git Workflow and CI](#git-workflow-and-ci)
8. [Troubleshooting](#troubleshooting)

---

## Project Overview

A0 Launcher is an Electron desktop shell that downloads UI content (`content.json`) from GitHub Releases at runtime and renders it locally. This enables shipping a stable executable while updating the UI/content independently via releases.

**Type**: Electron desktop app (packaged shell + downloadable content)
**Status**: Active development
**Primary languages**: JavaScript (CommonJS), HTML/CSS, YAML, Bash

Key constraints (non-negotiables) live in `.specify/memory/constitution.md`:
- Electron security isolation (treat downloaded content as untrusted)
- Shell/content contract (`content.json` schema)
- Release semver semantics (MAJOR builds executables; MINOR/PATCH reuse artifacts)

---

## Core Commands

### Development
- Install deps (recommended for CI/repro): `npm ci`
- Install deps (developer convenience): `npm install`
- Run dev mode (Electron GUI): `npm start`

### Build / Packaging
- Package (no installers): `npm run package`
- Make installers (current OS): `npm run make`
- Make installers (explicit platform):
  - macOS: `npm run make:mac`
  - Windows: `npm run make:win`
  - Linux: `npm run make:linux`

### macOS: Unsigned vs Signed/Notarized
- Unsigned local build (recommended for forks/dev): `SKIP_SIGNING=1 npm run make:mac`
- Signed + notarized (release-grade):
  - Requires secrets: `MACOS_CERT_P12`, `MACOS_CERT_PASSPHRASE`, `APPLE_ID`, `APPLE_PASSWORD`, `APPLE_TEAM_ID`
  - Control notarization:
    - Default: notarize when Apple credentials are present
    - Force on: `NOTARIZE=1 npm run make:mac`

### File-scoped "fast checks" (preferred)
There is no repo-standard JS test runner or linter script yet. Use these for fast feedback:
- Syntax check a file: `node --check shell/main.js`
- Syntax check a script: `node --check scripts/write-build-info.js`
- Validate Forge config loads: `node -e "require('./forge.config.js'); console.log('forge.config.js: OK')"`

### Testing
- **JavaScript tests**: No `npm test` script is currently defined in `package.json`.
- **SpecKit tooling tests**: exist under `.specify/tests/` (Python) but are not part of npm scripts.

---

## Project Structure

```
/
├── AGENTS.md                          # You are here
├── README.md                          # Human-facing overview + fork/testing + signing
├── package.json                       # npm scripts + Electron deps
├── package-lock.json                  # Lockfile (CI should use npm ci)
├── forge.config.js                    # Electron Forge config (makers, signing/notarization)
├── app/                               # Source UI content (bundled into content.json by CI)
│   └── index.html
├── shell/
│   ├── main.js                        # Electron main process (update/download/extract/load)
│   ├── preload.js                     # contextBridge surface (renderer API)
│   ├── loading.html                   # Loading UI shown before content loads
│   └── assets/                        # Icons + mac entitlements
├── scripts/
│   ├── write-build-info.js            # Generates shell/build-info.json from git remote/env
│   └── bootstrap-macos.sh             # Ephemeral macOS VM bootstrap (unsigned build/dev)
├── .github/workflows/
│   ├── bundle-content.yml             # Bundles app/ -> content.json and uploads
│   └── build.yml                      # Builds executables; reuses artifacts for minor/patch
├── specs/                              # SpecKit artifacts (may include not-yet-implemented work)
│   └── 001-docker-version-management/  # Example feature spec + contracts
├── .specify/                           # SpecKit memory/templates/scripts
└── .cursor/                            # Cursor commands and auto-generated rules
```

Key files worth opening first:
- `shell/main.js`: update and content lifecycle, Electron security settings
- `shell/preload.js`: renderer API surface (must stay minimal)
- `.github/workflows/build.yml`: release vs fork/dev build behavior

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agent0ai/a0-launcher](https://github.com/agent0ai/a0-launcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
