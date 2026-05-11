---
trigger: always_on
description: Lumiere is a cross-platform AI chat client built with **React Native** (Expo SDK 54) and **TypeScript**. It connects to 12+ AI providers (OpenClaw/Molt, Claude, OpenAI, OpenRouter, Gemini, Ollama, Kimi, Apple Intelligence, and more) through a unified provider abstraction. The app runs on **iOS, Android, Web, Desktop (Electron), and Chrome Extension**.
---

# Lumiere — AI Assistant Guide

## Overview

Lumiere is a cross-platform AI chat client built with **React Native** (Expo SDK 54) and **TypeScript**. It connects to 12+ AI providers (OpenClaw/Molt, Claude, OpenAI, OpenRouter, Gemini, Ollama, Kimi, Apple Intelligence, and more) through a unified provider abstraction. The app runs on **iOS, Android, Web, Desktop (Electron), and Chrome Extension**.

## Quick Reference

| Area             | Command / Path       |
| ---------------- | -------------------- |
| Install deps     | `pnpm install`       |
| Start dev server | `pnpm start`         |
| Run tests        | `pnpm test`          |
| Lint (auto-fix)  | `pnpm lint:fix`      |
| Format           | `pnpm format`        |
| Type check       | `npx tsc --noEmit`   |
| Build Chrome ext | `pnpm build:chrome`  |
| Build desktop    | `pnpm build:desktop` |

---

## Package Manager

Prefer **pnpm** over npm/yarn for all dependency management.

## Pre-Push Checks

Before every `git push`, you **must** run the following commands and fix any issues they report:

1. `pnpm lint:fix` — auto-fix ESLint issues, then resolve any remaining errors manually.
2. `pnpm format` — auto-format all files with Prettier.

Do **not** push code that has lint errors or formatting issues. If either command produces unfixable errors, address them in the code before pushing.

---

## Tech Stack

| Layer            | Technology                                            |
| ---------------- | ----------------------------------------------------- |
| Framework        | React Native 0.81, React 19, Expo 54, Expo Router 6   |
| Language         | TypeScript 5.9 (strict mode)                          |
| State management | Jotai (atoms with AsyncStorage persistence)           |
| Navigation       | Expo Router (file-based routing)                      |
| i18n             | i18next + react-i18next (11 languages)                |
| Styling          | React Native StyleSheet + custom theme system         |
| Animations       | React Native Reanimated 4                             |
| Testing          | Jest (jest-expo preset)                               |
| Linting          | ESLint 9 (flat config) + Prettier                     |
| CI/CD            | GitHub Actions, EAS Build, Fastlane, Cloudflare Pages |
| E2E testing      | Maestro                                               |

---

## Project Structure

```
lumiere/
├── app/                    # Expo Router file-based routes (22 screens)
│   ├── _layout.tsx         # Root layout: providers, auth gates, navigation
│   ├── index.tsx           # Home screen (main chat interface)
│   ├── settings.tsx        # Settings hub
│   ├── servers.tsx         # Server management
│   ├── sessions.tsx        # Session management
│   ├── missions.tsx        # Mission list (beta)
│   ├── create-mission.tsx  # Mission creation
│   ├── mission-detail.tsx  # Mission execution view
│   ├── scheduler.tsx       # Cron job management (Molt only)
│   ├── skills.tsx          # Skills marketplace (Molt only)
│   └── ...                 # Other screens (colors, language, triggers, etc.)
│
├── src/
│   ├── components/         # React components
│   │   ├── chat/           # ChatScreen, ChatInput, ChatMessage, voice UI
│   │   ├── missions/       # Mission planning and detail UI
│   │   ├── layout/         # Sidebar, layout wrappers
│   │   ├── ui/             # Reusable UI primitives (Button, Card, Badge, Text, etc.)
│   │   ├── gallery/        # Component showcase (dev only)
│   │   └── illustrations/  # SVG illustrations
│   │
│   ├── services/           # Backend integrations and business logic
│   │   ├── providers/      # ChatProvider abstraction layer
│   │   │   ├── types.ts    # ChatProvider interface, ProviderType, capabilities
│   │   │   ├── createProvider.ts  # Factory function
│   │   │   └── CachedChatProvider.ts  # Caching wrapper
│   │   ├── molt/           # OpenClaw WebSocket client (protocol v3)
│   │   ├── claude/         # Anthropic API client
│   │   ├── openai/         # OpenAI API client
│   │   ├── openrouter/     # OpenRouter SDK wrapper
│   │   ├── gemini/         # Google Gemini API client
│   │   ├── ollama/         # Local Ollama HTTP client
│   │   ├── kimi/           # Moonshot AI Kimi client
│   │   ├── apple-intelligence/  # iOS 18+ Foundation Models
│   │   ├── gemini-nano/    # Android on-device AI
│   │   ├── echo/           # Echo test provider
│   │   ├── clawhub/        # ClawHub API integration
│   │   ├── intents/        # Chat intent system
│   │   └── notifications/  # Push/background notifications
│   │
│   ├── hooks/              # Custom React hooks (~35 hooks)
│   │   ├── useChatProvider.ts       # Provider lifecycle management
│   │   ├── useServers.ts            # Server CRUD + secure token storage
│   │   ├── useSlashCommands.ts      # 38 built-in slash commands
│   │   ├── useMissionList.ts        # Mission data (read-only)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ajamaica/lumiere](https://github.com/ajamaica/lumiere) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
