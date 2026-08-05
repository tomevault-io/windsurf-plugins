---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Git Commit Rules

- Git author is already configured for this repo, do not modify git config
- **NEVER** add `Co-Authored-By` lines in commit messages
- Push to GitHub remote (`github`) only, ignore `origin` (deprecated GitLab)

## Project Workflow Triggers

When the user asks to do any of the following in this repository:

- publish to the LumenX GitHub mirror
- run the LumenX GitHub publish workflow
- follow the LumenX GitHub release or PR flow
- prepare a GitHub-safe branch, commit, push, or PR for LumenX
- use `/lumenx-git-publish`

Treat that as a request to load and follow:

`.codex/workflows/lumenx-git-publish.md`

When the user asks to do any of the following in this repository:

- onboard a new model into LumenX
- update model docs, model versions, defaults, or parameters
- refresh Wan / Kling / Vidu / PixVerse model support
- run the LumenX model onboarding workflow
- review whether a model change is catalog-only or also needs runtime / UI work
- use `/lumenx-model-onboarding`

Treat that as a request to load and follow:

`.codex/workflows/lumenx-model-onboarding.md`

When the user asks to do any of the following in this repository:

- build the LumenX desktop app
- package LumenX Studio for macOS or Windows
- create a DMG or EXE build
- run the LumenX desktop build workflow
- use `/lumenx-build`

Treat that as a request to load and follow:

`.codex/workflows/lumenx-build.md`

This repository does not rely on native slash commands in Codex. The strings `/lumenx-git-publish`, `/lumenx-build`, and `/lumenx-model-onboarding` are textual aliases for the workflows above.

## Workflow Files

- `.claude/commands/lumenx-git-publish.md` remains the Claude project command source.
- `.claude/commands/lumenx-build.md` remains the Claude project command source.
- `.claude/commands/lumenx-model-onboarding.md` remains the Claude project command source.
- `.codex/workflows/lumenx-git-publish.md` is the Codex workflow mirror for the same project process.
- `.codex/workflows/lumenx-build.md` is the Codex workflow mirror for the desktop build process.
- `.codex/workflows/lumenx-model-onboarding.md` is the Codex workflow mirror for model onboarding, catalog updates, and verification.

If both Claude and Codex guidance exist, preserve behavior parity unless the user asks for divergence.

# AI Comic Generator Platform

## Overview

The AI Comic Generator is a complete AI-powered comic video production platform that supports the full workflow from script to finished video. It uses Next.js frontend with FastAPI backend, integrating AI services like Qwen from Alibaba Cloud.

## Architecture

### Frontend
- Framework: Next.js 14 + React 18 + TypeScript + Tailwind CSS
- State management: Zustand
- HTTP client: Axios
- 3D rendering: Three.js + @react-three/fiber
- Animation: Framer Motion

### Backend
- Framework: FastAPI (Python 3.11+)
- AI integration: Alibaba Cloud Qwen/Wanx services via DashScope
- Data validation: Pydantic
- File storage: Local + Alibaba Cloud OSS

### Core Components

#### Frontend Structure
```
frontend/
├── src/app/              # Next.js App Router pages
├── src/components/       # React components
│   ├── layout/          # Layout components
│   ├── modules/         # Feature modules (ScriptInput, ArtDirection, etc.)
│   ├── canvas/          # Canvas-related components
│   └── project/         # Project-specific components
├── src/lib/             # Utilities (API client at api.ts)
└── src/store/           # Zustand stores
```

#### Backend Structure
```
src/
├── apps/comic_gen/      # Core comic generation logic
│   ├── api.py           # FastAPI routes (main entry point)
│   ├── pipeline.py      # Core business flow management
│   ├── models.py        # Data models (Pydantic)
│   ├── llm.py           # LLM interaction (script analysis, etc.)
│   ├── assets.py        # Asset generation (characters/scenes/props)
│   ├── storyboard.py    # Storyboard generation
│   ├── video.py         # Video generation
│   ├── audio.py         # Audio generation
│   └── export.py        # Video export/synthesis
├── models/              # AI model wrappers
├── utils/               # Utility functions (OSS integration)
└── config.py            # Global configuration
```

## Development Commands

### Initial Setup
```bash
# Copy environment template
cp .env.example .env
# Edit .env and add your Alibaba Cloud API keys
```

### Backend Development
```bash
# Install dependencies
pip install -r requirements.txt

# Create output directories
mkdir -p output/uploads

# Start backend server
./start_backend.sh
# or
python -m uvicorn src.apps.comic_gen.api:app --reload --host 0.0.0.0 --port 17177

# API docs available at: http://localhost:17177/docs
```

### Frontend Development
```bash
cd frontend
npm install
npm run dev
# Frontend available at: http://localhost:3008
```

### Full Development Mode
```bash
# Terminal 1: Start backend
./start_backend.sh

# Terminal 2: Start frontend
cd frontend && npm run dev
```

### Desktop App Mode
```bash
# Run the complete desktop application
python main.py
```

## File Structure

### Output Management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brokenmoonbeam/lumenx-comfyui](https://github.com/brokenmoonbeam/lumenx-comfyui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
