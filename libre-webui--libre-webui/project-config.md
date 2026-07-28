---
trigger: always_on
description: > **Project:** Libre WebUI — Privacy-first, open-source AI chat interface
---

# Libre WebUI — Agent Reference

> **Project:** Libre WebUI — Privacy-first, open-source AI chat interface
> **Version:** 0.9.0
> **License:** Apache 2.0
> **Maintainer:** Kroonen AI, Inc. + open-source community
> **Repository:** https://github.com/libre-webui/libre-webui

---

## 1. Project Overview

Libre WebUI is a **self-hosted AI chat interface** that connects to Ollama (local) and 10+ cloud providers (OpenAI, Anthropic, Google, etc.) via a plugin system. It runs as:

- A web app (React + Express)
- An Electron desktop app (macOS, Windows, Linux)
- A Docker container (with or without bundled Ollama)
- A Homebrew package
- A Kubernetes deployment (Helm chart)

**Core philosophy:** Zero telemetry. No tracking. Apache 2.0 forever. Local-first.

---

## 2. Architecture

```
┌──────────────────────────────────────────────────────────┐
│                    Libre WebUI                           │
├──────────────────────┬───────────────────────────────────┤
│   Frontend           │   Backend                         │
│   React 18 + TS      │   Express 5 + TS                  │
│   Vite 8             │   better-sqlite3                  │
│   Zustand (state)    │   AES-256-GCM encryption          │
│   TanStack Query     │   WebSocket streaming             │
│   i18next (25 langs) │   JWT auth                        │
│   Tailwind CSS       │   Helmet + rate limiting          │
│   Framer Motion      │                                   │
│   Lucide icons       │                                   │
│   React Markdown     │                                   │
│   KaTeX (math)       │                                   │
├──────────────────────┴───────────────────────────────────┤
│              Plugin Layer (JSON config files)            │
│   Ollama │ OpenAI │ Anthropic │ Google │ Groq │ …       │
├──────────────────────────────────────────────────────────┤
│   Electron (Desktop) │ Docker │ Kubernetes │ npx CLI     │
└──────────────────────────────────────────────────────────┘
```

---

## 3. Monorepo Structure

```
libre-webui/
├── package.json              # Root workspace manifest
├── package-lock.json
├── .npmrc                    # legacy-peer-deps=true
│
├── frontend/                 # npm workspace: libre-webui-frontend
│   ├── package.json
│   ├── vite.config.ts
│   ├── tailwind.config.js
│   ├── postcss.config.js
│   ├── tsconfig.json
│   └── src/
│       ├── main.tsx           # App entry point
│       ├── App.tsx            # Root component with routing
│       ├── index.css          # Global styles + Tailwind
│       ├── layouts/
│       │   └── ChatLayout.tsx # Main chat layout wrapper
│       ├── pages/
│       │   ├── ChatPage.tsx   # Main chat page
│       │   ├── LoginPage.tsx  # Auth login page
│       │   ├── PersonasPage.tsx
│       │   ├── ModelsPage.tsx
│       │   ├── GalleryPage.tsx
│       │   └── UserManagementPage.tsx
│       ├── components/        # ~59 components
│       │   ├── ChatInput.tsx
│       │   ├── ChatMessage.tsx
│       │   ├── ChatMessages.tsx
│       │   ├── Sidebar.tsx
│       │   ├── SettingsModal.tsx
│       │   ├── ModelSelector.tsx
│       │   ├── PersonaCard.tsx / PersonaForm.tsx / PersonaManager.tsx
│       │   ├── PluginManager.tsx
│       │   ├── ImageGenerationPanel.tsx
│       │   ├── TTSButton.tsx
│       │   ├── ArtifactRenderer.tsx / ArtifactContainer.tsx
│       │   ├── LoginForm.tsx / SignupForm.tsx / FirstTimeSetup.tsx
│       │   ├── UserMenu.tsx / UserManager.tsx
│       │   ├── ProtectedRoute.tsx
│       │   ├── HuggingFaceModelBrowser.tsx
│       │   ├── ThemeToggle.tsx
│       │   ├── LanguageSwitcher.tsx
│       │   └── ui/            # Reusable UI primitives
│       ├── store/
│       │   ├── appStore.ts    # Global app state (Zustand)
│       │   ├── chatStore.ts   # Chat sessions & messages
│       │   ├── authStore.ts   # Auth state & JWT
│       │   └── pluginStore.ts # Plugin state
│       ├── hooks/
│       │   ├── useChat.ts           # Chat streaming logic
│       │   ├── useInitializeApp.ts  # App init logic
│       │   └── useKeyboardShortcuts.ts
│       ├── services/
│       │   └── userService.ts
│       ├── utils/
│       │   ├── api.ts               # API client (Axios)
│       │   ├── websocket.ts         # WebSocket client
│       │   ├── artifactParser.ts    # Artifact HTML/SVG parsing
│       │   ├── config.ts            # App config
│       │   └── oauthCallback.ts     # OAuth flow handling
│       ├── types/
│       │   └── index.ts             # Shared TS types
│       ├── i18n/
│       │   ├── index.ts
│       │   └── locales/             # 25+ language JSON files
│       │       ├── en.json, fr.json, de.json, es.json, …
│       └── assets/
│
├── backend/                  # npm workspace: libre-webui-backend
│   ├── package.json
│   ├── tsconfig.json
│   └── src/
│       ├── index.ts           # Express app entry (4400+ lines)
│       ├── env.ts             # Environment variable loading
│       ├── db.ts              # SQLite initialization & schema
│       ├── storage.ts         # Storage abstraction (SQLite/JSON)
│       ├── middleware/
│       │   ├── index.ts       # Error handlers, request logger
│       │   └── auth.ts        # JWT auth, role checks
│       ├── routes/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [libre-webui/libre-webui](https://github.com/libre-webui/libre-webui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
