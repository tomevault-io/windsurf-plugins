---
trigger: always_on
description: This file is the single source of project guidance for AI coding agents (Claude Code, Codex, etc.) and human contributors working in this repository. Its scope covers the entire repo. For repository-wide policies and detailed workflows, defer to the canonical guidance in `CONTRIBUTING.md`.
---

# AGENTS.md

This file is the single source of project guidance for AI coding agents (Claude Code, Codex, etc.) and human contributors working in this repository. Its scope covers the entire repo. For repository-wide policies and detailed workflows, defer to the canonical guidance in `CONTRIBUTING.md`.

## Project overview

`streamlit-webrtc` is a Streamlit component that enables real-time video and audio processing using WebRTC. The library lets developers build web apps that capture, process, and stream video/audio data in real-time directly in the browser.

## Architecture

### Core modules

- `streamlit_webrtc/component.py` — main Streamlit component interface (`webrtc_streamer`).
- `streamlit_webrtc/webrtc.py` — WebRTC connection management and worker processes.
- `streamlit_webrtc/models.py` — data models and base classes for processors.
- `streamlit_webrtc/process.py` — audio/video processing tracks and transformations.
- `streamlit_webrtc/factory.py` — factory functions for creating processing tracks.

### Frontend

- React + TypeScript under `streamlit_webrtc/frontend/`.
- Vite for bundling and dev.
- Material-UI for components.
- WebRTC adapter for cross-browser compatibility.

### Key abstractions

- **Processors** — `VideoProcessorBase` and `AudioProcessorBase` for frame-by-frame processing.
- **Tracks** — `MediaStreamTrack` implementations for audio/video streams.
- **WebRtcWorker** — manages WebRTC connections and media processing.
- **Component context** — `WebRtcStreamerContext` for accessing stream state.

### Processing flow

1. Browser captures media via WebRTC.
2. Frames are sent to the Python backend.
3. Custom processors transform frames (video/audio).
4. Processed frames are sent back to the browser.
5. Browser displays/plays the processed media.

## Quickstart

- Install dependencies with `uv sync`.
- Install Git hooks: `pre-commit install`.
- Use `make format` before committing to format both backend and frontend code.

## Running the project

For local frontend development:

- Set `_RELEASE = False` in `streamlit_webrtc/component.py` (do **not** commit this change). When `_RELEASE = False`, the component loads the dev build served by Vite. The build commands fail while `_RELEASE = False` is set — see the `build` rule in `Makefile` and `release_check.py`.
- Start the frontend dev server:
  - `cd streamlit_webrtc/frontend && pnpm dev`
- In another terminal, run the app:
  - `streamlit run home.py`

The frontend dev server runs on a different port than the production build.

## Testing

- Backend: `uv run pytest` (tests live in `tests/`).
- Frontend: `cd streamlit_webrtc/frontend && pnpm test` (Vitest + React Testing Library).
- Manual: example apps in `pages/`.

## Type checking

- `uv run mypy .` for backend typing.

## Code quality

- Format backend: `uv run ruff format . && uv run ruff check . --fix` (or `make format/backend`).
- Format frontend: `cd streamlit_webrtc/frontend && pnpm format` (or `make format/frontend`).
- Format everything: `make format`.

## Building

- Full build (frontend + backend): `make build`.
- Frontend-only build: `cd streamlit_webrtc/frontend && pnpm run build`.

## WebRTC configuration

- STUN/TURN servers are configured via the `rtc_configuration` argument to `webrtc_streamer()`.
- The default is Google's public STUN server.
- Production deployments may need a TURN server (e.g., Twilio).

## Changelog fragments (required for every PR)

- **Every PR must include a changelog fragment under `changelog.d/`.** This is mandatory — the release pipeline collects fragments to build the next release's changelog, and a PR without one will not be reflected in the release notes.
- Create one with `uv run scriv create --edit`. It writes a timestamped file (e.g. `changelog.d/20260513_042424_<user>_<slug>.md`) prefilled with the available categories.
- Pick **one** category and delete the rest:
  - `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, `Security` — for user-visible changes.
  - `Chore` — for internal changes that don't affect users (refactors, dead-code cleanup, tooling, CI, build infra, internal docs).
- Write a single bullet describing the change. For user-visible categories, write it from the user's perspective; for `Chore`, from the maintainer's. Reference the issue or PR where it adds context.
- Commit the fragment as part of the same PR — not as a follow-up.
- CI opens a changelog preview PR from committed fragments; maintainers merge it before the automated release build runs.

## Contribution notes

- Keep changes minimal and focused; prefer small, reviewable commits.
- Follow existing code patterns and naming conventions in each area.

## OSS issue automation rules

This repository uses GitHub Actions to let Claude handle initial issue triage and a portion of the implementation work. When an agent works from an issue, follow these rules:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whitphx/streamlit-webrtc](https://github.com/whitphx/streamlit-webrtc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
