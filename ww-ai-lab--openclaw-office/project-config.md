---
trigger: always_on
description: > [中文版](./AGENTS.zh.md)
---

# OpenClaw Office — Agent Development Guide

> [中文版](./AGENTS.zh.md)

This document provides context and rules for AI coding assistants (Codex, Claude, Cursor Agent, etc.) working on this project.

## Project Overview

OpenClaw Office is the visual monitoring and management frontend for the [OpenClaw](https://github.com/openclaw/openclaw) Multi-Agent system. It connects to the OpenClaw Gateway via WebSocket to visualize Agent collaboration as a "digital office" and provides a full console management interface.

## Tech Stack

| Category         | Technology                                  |
| ---------------- | ------------------------------------------- |
| Language         | TypeScript (ESM, strict mode)               |
| UI Framework     | React 19                                    |
| Build Tool       | Vite 6                                      |
| State Management | Zustand 5 + Immer                           |
| Styling          | Tailwind CSS 4                              |
| 2D Rendering     | SVG + CSS Animations                        |
| Routing          | React Router 7                              |
| Charts           | Recharts                                    |
| i18n             | i18next + react-i18next                     |
| Testing          | Vitest + @testing-library/react             |
| Real-time        | Native WebSocket API                        |

## Feature Modules

### Office View (`/`)

Isometric-style 2D/3D virtual office for real-time Agent status visualization:

- **2D Floor Plan** — SVG office scene with desks, furniture (desk/chair/sofa/plant/coffee cup), Agent avatars, and status animations
- **Agent Avatars** — Deterministically generated SVG avatars from agentId with status indicators (idle/working/speaking/error)
- **Collaboration Lines** — Visual connections for inter-Agent messaging
- **Bubble Panels** — Markdown text stream and tool call display
- **Side Panels** — Agent details, metrics charts (Token line chart/cost pie chart/activity heatmap), SubAgent graphs, event timeline

### Chat

Bottom-docked chat bar for real-time Agent conversations:

- Agent selector, streaming message display, Markdown rendering
- Chat history drawer, send/abort controls

### Console (`/dashboard`, `/agents`, `/channels`, `/skills`, `/cron`, `/settings`)

Full system management interface:

- **Dashboard** — Overview stat cards, alert banners, Channel/Skill overview, quick navigation
- **Agents** — Agent list/create/delete, detail tabs (Overview/Channels/Cron/Skills/Tools/Files)
- **Channels** — Channel cards, config dialogs, stats, WhatsApp QR binding flow
- **Skills** — Skill marketplace, install options, skill details
- **Cron** — Scheduled task management and stats bar
- **Settings** — Provider management (add/edit/model editor), appearance/Gateway/developer/advanced/about/update

## Directory Structure

```
src/
├── main.tsx / App.tsx          # Entry point and routing
├── i18n/                       # Internationalization (zh/en)
├── gateway/                    # Gateway communication layer
│   ├── ws-client.ts / ws-adapter.ts  # WebSocket client + auth + reconnect
│   ├── rpc-client.ts           # RPC request wrapper
│   ├── event-parser.ts         # Event parsing + state mapping
│   ├── adapter.ts / adapter-provider.ts  # Adapter pattern (real/mock switch)
│   └── mock-adapter.ts         # Mock mode adapter
├── store/                      # Zustand stores
│   ├── office-store.ts         # Main store (Agent state, connection, UI)
│   ├── agent-reducer.ts / metrics-reducer.ts / meeting-manager.ts
│   └── console-stores/         # Per-page console stores
├── components/
│   ├── layout/                 # AppShell / ConsoleLayout / Sidebar / TopBar
│   ├── office-2d/              # 2D SVG floor plan + furniture components
│   ├── office-3d/              # 3D R3F scene
│   ├── overlays/               # SpeechBubble and other HTML overlays
│   ├── panels/                 # Detail/metrics/chart panels
│   ├── chat/                   # Chat dock bar components
│   ├── console/                # Console feature page components
│   │   ├── dashboard/ / agents/ / channels/
│   │   ├── skills/ / cron/ / settings/ / shared/
│   │   └── ...
│   ├── pages/                  # Console route pages
│   └── shared/                 # Shared components (Avatar/LanguageSwitcher etc.)
├── hooks/                      # Custom React hooks
├── lib/                        # Utility library
└── styles/                     # Global styles
```

## Development Commands

```bash
pnpm install              # Install dependencies
pnpm dev                  # Start dev server (port 5180)
pnpm build                # Production build
pnpm test                 # Run tests
pnpm test:watch           # Test watch mode
pnpm typecheck            # TypeScript type check
pnpm lint                 # Oxlint check
pnpm format               # Oxfmt format
pnpm check                # lint + format check
```

## Coding Standards

- TypeScript strict mode; **no `any`**
- Files must not exceed 500 lines; split when longer
- Components: PascalCase; hooks: useCamelCase
- Oxlint + Oxfmt formatting (consistent with OpenClaw main project)
- Comments only for non-obvious logic

## OpenClaw Gateway Integration

### Connection & Authentication


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WW-AI-Lab/openclaw-office](https://github.com/WW-AI-Lab/openclaw-office) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
