---
trigger: always_on
description: Multistream is a native, cross-platform desktop application that enables power users to watch multiple live streams (Twitch, Kick, YouTube) simultaneously, featuring an integrated real-time chat interface.
---

# Multistream

## Overview

Multistream is a native, cross-platform desktop application that enables power users to watch multiple live streams (Twitch, Kick, YouTube) simultaneously, featuring an integrated real-time chat interface.

### Core Objectives & Philosophy

- **Privacy by Design:** 100% local processing. No middleman servers, no data collection, and no telemetry. Never introduce third-party tracking or cloud analytics.
- **Direct Connections:** Streams and chat use direct connections (e.g., standard iframes, WebSocket) to official platforms.
- **Lightweight & Performant:** Built with Tauri and Rust to maintain incredibly low memory usage and high performance compared to traditional Electron apps. Keep dependencies minimal.
- **Local AI Transcription:** Real-time translation and transcription must remain 100% local, offline, and free, powered by Whisper.cpp on the CPU. Do not integrate paid cloud AI APIs for this feature.

## Hard Rules

- ALMOST NEVER write comments. We're senior engineers here, not learners.
- NEVER run the backend or frontend manually. The human is already doing this.
- ALWAYS test rust backend changes by running 'cargo check', and if you modify any business logic or important commands, also run 'cargo test'.
- ALWAYS test frontend changes by running Playwright MCP.
- ALWAYS follow the current design system and minimalist aesthetics of the application. Do not invent new visual patterns, do not introduce jarring colors, and strictly respect the dark/neutral color palette (e.g., bg-[#0f1115], text-gray-400) used across the app.
- ALWAYS profile before suggesting architectural performance changes. Do not recommend virtualization, caching, memoization, background workers, or other advanced optimizations unless there is evidence they address the actual bottleneck.
- NEVER write a text hardcoded ALWAYS write following i18n pattern (on frontend)
- ALWAYS validate changes according to scope: UI/interface changes require running 'bun run desktop:typecheck'; logic/composable changes require running both 'bun run desktop:typecheck' and unit tests.

## Local Agent Skills

This repository contains custom, specialized skills for AI Agents located in the `.agents/skills/` directory. **ALWAYS** invoke and read these local skills when working on the respective domains:

- **[`multistream-desktop-frontend`](.agents/skills/desktop-frontend/SKILL.md)**: Pragmatic architecture guide, TypeScript types, Composables, Vue 3 components, i18n, and tests for the desktop app (`apps/desktop/src/`).
- **[`multistream-desktop-backend`](.agents/skills/desktop-backend/SKILL.md)**: Pragmatic architecture guide, IPC command patterns, Tokio async concurrency, error handling, security, and unit testing guidelines for the Multistream Tauri 2 Rust backend in `apps/desktop/src-tauri/`.
- **[`multistream-website`](.agents/skills/website/SKILL.md)**: Pragmatic architecture guide, Astro SSG setup, Tailwind CSS v4 styling, i18n, Deep Link Gateway, and dynamic GitHub release links for the website (`apps/website/`).
- **[`multistream-desktop-frontend-testing`](.agents/skills/desktop-frontend-testing/SKILL.md)**: Comprehensive guide, AAA pattern, Tauri IPC mocking, fake timers, and best practices for writing unit tests in `apps/desktop/src/`.
- **[`multistream-graveyard`](.agents/skills/graveyard/SKILL.md)**: Explains the Multistream Graveyard mechanism, how it works, and why it is necessary to prevent WebView IPC crashes.
- **[`multistream-adding-language`](.agents/skills/adding-language/SKILL.md)**: Guides agents and developers through the exact process of adding a new language to the Multistream application.
- **[`multistream-critical-edge-case-analysis`](.agents/skills/critical-edge-case-analysis/SKILL.md)**: Rigorous, egoless methodology for auditing business logic, identifying hidden edge cases, questioning assumptions, remediating logic flaws, and authoring bulletproof unit tests (AAA pattern) across Multistream.

## Tech Stack

- **Frontend Framework:** [Vue 3](https://vuejs.org/)
- **Desktop Framework:** [Tauri 2](https://v2.tauri.app/)
- **Language:** TypeScript, Rust
- **Styling:** [Tailwind CSS](https://tailwindcss.com/)
- **Tooling/Runtime:** Vite, Bun

## Development

### Commands

| Action                  | Command                              |
| :---------------------- | :----------------------------------- |
| **Dev (Tauri)**         | `bun run desktop:tauri:dev`          |
| **Build (Tauri)**       | `bun run desktop:build:tauri`        |
| **Tests (Unit)**        | `bun run desktop:test`               |
| **Tests (Unit Single)** | `bun run desktop:test -- <filename>` |
| **Coverage**            | `bun run desktop:test:coverage`      |
| **Tests (E2E)**         | `bun run desktop:test:e2e`           |
| **Tests (UI)**          | `bun run desktop:test:e2e:ui`        |
| **Type Check**          | `bun run desktop:typecheck`          |
| **Linting**             | `bun run lint`                       |

## Architecture & Conventions

- The project structure is split into three main parts:
  - `apps/desktop/src/`: Vue frontend for the Desktop App.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ilanzgx/multistream](https://github.com/ilanzgx/multistream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
