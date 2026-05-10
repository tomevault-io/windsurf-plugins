---
trigger: always_on
description: This document provides guidance for AI agents working on the NAM-BOT project.
---

# AGENTS.md - NAM-BOT Development Guide (v0.5.1-rc.1)

This document provides guidance for AI agents working on the NAM-BOT project.

---

## 0. Working with Contributors

- Assume some contributors are new to Electron or desktop-app tooling; explain concepts in plain language, especially around the Electron build, development, and packaging workflow.
- Spell out the exact npm scripts or commands (and their purpose) when you reference them so future threads understand how to start the dev server and package the app from scratch.

### 0.1 Project-Local Agent Skills

- Project-local agent workflow files live under `.agents/`.
- Prefix repo-scoped skill names with `nam-` so they stay easy to distinguish from global skills.
- Use `.agents/skills/nam-release-workflow/SKILL.md` when the user asks to update the changelog, choose a version bump, clean generated release trash, commit, or push.

### 0.2 Documentation Discipline

- When a significant change is made to a core feature, update the relevant documentation in `docs/` as part of the same work whenever practical.
- Core features include, at minimum, Jobs, Presets, Settings, Diagnostics, Dashboard, and Setup Guide.
- If no matching document exists yet, create one rather than leaving the feature undocumented.

---

## 1. Project Overview

NAM-BOT is a desktop training front-end for Neural Amp Modeler (NAM). Built with Electron + electron-vite + React + TypeScript, it provides a modern UI for managing NAM training jobs with queueing, presets, and diagnostics.

**Tech Stack:**

- Electron 40.x with electron-vite
- React 19 + TypeScript
- Zustand for state management
- electron-log for logging
- electron-builder for packaging

---

## 2. Build Commands

```bash
# Development - runs app in development mode with hot reload
npm run dev

# Production build - builds all three targets (main, preload, renderer)
npm run build

# Preview production build
npm run preview

# Package Windows installer (builds first, then packages)
npm run package
```

## 2.1 GitHub Actions Release Flow

- `.github/workflows/ci.yml` runs on every push and pull request and should be treated as the baseline build-health check.
- `.github/workflows/release.yml` is for distributable Windows and macOS releases and should only publish when a Git tag matching `v*` is pushed, for example `v0.2.5`, or when manually triggered with `workflow_dispatch`.
- `.github/workflows/preview-release.yml` publishes rolling prerelease preview builds for pushes to `main`; those preview entries should not be treated as stable releases.
- When explaining release flow to contributors, spell out that ordinary pushes do **not** create GitHub Releases automatically.
- Preferred timing: push the finished commit to `main`, do the final smoke test, and only then push the version tag that should publish publicly.
- When an agent pushes `main` for release work, it should always mention whether a matching version tag should also be pushed for a full GitHub release.
- Agents must never push a release tag automatically without an explicit user confirmation in that thread, even when the version bump and release commit are already prepared.
- Release tags publish Windows installer, portable ZIP, and macOS DMG assets.
- GitHub Release notes are generated from the matching `CHANGELOG.md` version section, so keep release entries user-facing and concise before pushing the `v*` tag.
- Preferred release trigger example:

```bash
git tag v0.3.4
git push origin v0.3.4
```

**Build Output:**

- `out/` - Compiled JavaScript
- `release/` - Packaged installers
  - `NAM-BOT-Setup-1.0.0.exe` - NSIS installer
  - `win-unpacked/NAM-BOT.exe` - Portable executable

---

## 3. Project Structure

```text
nam-bot/
├── src/
│   ├── main/                    # Electron main process
│   │   ├── index.ts            # Entry point, window creation
│   │   ├── backend/            # NAM backend adapter
│   │   ├── config/             # Job config builder
│   │   ├── ipc/                # IPC handlers
│   │   ├── jobs/               # Queue manager
│   │   ├── persistence/        # Settings/data stores
│   │   └── types/              # TypeScript types
│   ├── preload/
│   │   └── index.ts            # Secure contextBridge API
│   └── renderer/               # React UI
│       ├── App.tsx             # Main app component
│       ├── features/           # Screen components
│       ├── state/              # Zustand stores
│       └── styles/             # CSS
├── electron.vite.config.ts
├── electron-builder.yml
├── tsconfig.json
└── package.json
```

---

## 4. Code Style Guidelines

### 4.1 TypeScript

- **Always use explicit types** for function parameters and return types
- **Use strict mode** - no `any` types
- **Use interfaces** for object shapes, types for unions
- **Avoid `as` casts** - use proper type guards instead

### 4.2 Imports

- **Order imports**: external → internal → relative
- **Use named imports** for React and libraries

```typescript
import { useState, useEffect } from 'react'
import { useAppStore } from './state/store'
import { JobSpec } from '../types/jobs'
import { validateBackend } from '../backend/adapter'
```

### 4.3 Naming Conventions

| Type               | Convention | Example             |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daveotero/NAM-BOT](https://github.com/daveotero/NAM-BOT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
