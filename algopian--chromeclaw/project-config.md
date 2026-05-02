---
trigger: always_on
description: ChromeClaw is a Chrome extension that provides AI chat in the browser's side panel with multi-provider LLM support. Built with React 19, TypeScript, and pi-mono (`@mariozechner/pi-ai` + `@mariozechner/pi-agent-core`). Users add their own API keys — no login or proxy required.
---

# CLAUDE.md — ChromeClaw Extension

## What is this project?

ChromeClaw is a Chrome extension that provides AI chat in the browser's side panel with multi-provider LLM support. Built with React 19, TypeScript, and pi-mono (`@mariozechner/pi-ai` + `@mariozechner/pi-agent-core`). Users add their own API keys — no login or proxy required.

## Monorepo layout

Flat monorepo orchestrated with **Turborepo** (`turbo.json`).

```
chromeclaw/
├── chrome-extension/              # Background service worker, manifest
│   └── src/background/
│       ├── agents/                # Agent personas, model adapter, stream handler
│       ├── channels/              # Telegram + WhatsApp messaging bridges
│       ├── context/               # System prompt assembly, context compaction
│       ├── cron/                  # Scheduled task runner
│       ├── errors/                # Error handling
│       ├── logging/               # Logging utilities
│       ├── media-understanding/   # Speech-to-text, media transcription
│       ├── memory/                # BM25 + embedding hybrid search, memory journal
│       ├── tools/                 # All tool implementations
│       └── tts/                   # Text-to-speech (OpenAI, Kokoro)
├── pages/
│   ├── side-panel/                # Primary chat UI (overlay sidebar mode)
│   ├── full-page-chat/            # Full-page chat (push sidebar mode)
│   ├── offscreen-channels/        # Offscreen page for channel message handling
│   └── options/                   # Settings page (tabbed, see below)
├── packages/
│   ├── baileys/                   # WhatsApp (Baileys) integration
│   ├── config-panels/             # Options page tab panels and tab group definitions
│   ├── dev-utils/                 # Dev utilities
│   ├── env/                       # Build-time CEB_* environment variables
│   ├── hmr/                       # Hot module reload for extension dev
│   ├── i18n/                      # Internationalization
│   ├── module-manager/            # Module dependency management CLI
│   ├── shared/                    # Types, hooks (useLLMStream), prompts, env config
│   ├── skills/                    # Skill template loading and parsing
│   ├── storage/                   # Chrome storage + IndexedDB (Dexie.js) — all persistence
│   ├── tailwindcss-config/        # Tailwind configuration
│   ├── tsconfig/                  # Base TypeScript configs
│   ├── ui/                        # React components (shadcn/ui + custom chat components)
│   ├── vite-config/               # Shared Vite configuration
│   └── zipper/                    # Extension ZIP packaging
├── tests/playwright/              # E2E tests
├── bash-scripts/                  # Shell scripts (copy-env, set-global-env, update-version)
├── docs/                          # Documentation
├── turbo.json                     # Turborepo config
├── vitest.config.ts               # Shared Vitest config
└── playwright.config.ts           # Playwright config
```

## Commands

All commands run from the **repo root**:

```bash
pnpm install          # Install deps (postinstall copies .env from .example.env)
pnpm dev              # Watch mode with HMR
pnpm dev:firefox      # Watch mode targeting Firefox
pnpm build            # Production build → dist/
pnpm build:firefox    # Production build for Firefox
pnpm test             # Vitest unit tests
pnpm test:watch       # Vitest watch mode
pnpm test:coverage    # Vitest with coverage
pnpm test:e2e         # Playwright E2E tests (requires prior build)
pnpm lint             # ESLint (flat config)
pnpm lint:fix         # ESLint with auto-fix
pnpm format           # Prettier write
pnpm format:check     # Prettier check
pnpm type-check       # TypeScript strict check
pnpm quality          # lint + format:check + type-check + test
pnpm zip              # Build + package as ZIP
pnpm zip:firefox      # Build Firefox + package as ZIP
pnpm clean            # Clean bundles, turbo cache, and node_modules
pnpm clean:install    # Clean node_modules + fresh install
pnpm update-version   # Update version in manifest + package.json
pnpm module-manager   # Module dependency management CLI
pnpm prepare          # Husky git hooks setup
```

## Architecture

### Data flow
```
Side Panel / Full-Page Chat
  → useLLMStream hook (chrome.runtime.Port)
  → Background Service Worker (llm-stream.ts)
  → Model Adapter (chatModelToPiModel) → pi-mono streamSimple()
  → LLM Provider (OpenAI/Anthropic/Google/OpenRouter/Custom/Local)
  → SSE stream back through Port → UI updates
```

### Storage
- **Chrome storage (local/session)**: Settings, tool configs
- **IndexedDB via Dexie.js** (`chromeclaw` database, v13): agents, chats, messages, artifacts, workspaceFiles, memoryChunks, scheduledTasks, taskRunLogs, embeddingCache
- Models are also stored in IndexedDB (`DbChatModel` type)

### Key components
- **Background SW** (`chrome-extension/src/background/`): LLM streaming with tool calling, context compaction, memory search, auto-titling, channels, cron, TTS, media understanding

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [algopian/chromeclaw](https://github.com/algopian/chromeclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
