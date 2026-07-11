---
trigger: always_on
description: Project directory structure overview
---


# LobeChat Project Structure

## Directory Structure

note: some files are not shown for simplicity.

```plaintext
lobe-chat/
├── apps/                           # Applications directory
│   └── desktop/                    # Electron desktop application
│       ├── src/                    # Desktop app source code
│       └── resources/              # Desktop app resources
├── docs/                           # Project documentation
│   ├── development/                # Development docs
│   ├── self-hosting/               # Self-hosting docs
│   └── usage/                      # Usage guides
├── locales/                        # Internationalization files (multiple locales)
│   ├── en-US/                      # English (example)
│   └── zh-CN/                      # Simplified Chinese (example)
├── packages/                       # Monorepo packages directory
│   ├── const/                      # Constants definition package
│   ├── database/                   # Database related package
│   ├── electron-client-ipc/        # Electron renderer ↔ main IPC client
│   ├── electron-server-ipc/        # Electron main process IPC server
│   ├── model-bank/                 # Built-in model presets/catalog exports
│   ├── model-runtime/              # AI model runtime package
│   ├── types/                      # TypeScript type definitions
│   ├── utils/                      # Utility functions package
│   ├── file-loaders/               # File processing packages
│   ├── prompts/                    # AI prompt management
│   └── web-crawler/                # Web crawling functionality
├── public/                         # Static assets
│   ├── icons/                      # Application icons
│   ├── images/                     # Image resources
│   └── screenshots/                # Application screenshots
├── scripts/                        # Build and tool scripts
├── src/                            # Main application source code (see below)
├── .cursor/                        # Cursor AI configuration
├── docker-compose/                 # Docker configuration
├── package.json                    # Project dependencies
├── pnpm-workspace.yaml            # pnpm monorepo configuration
├── next.config.ts                  # Next.js configuration
├── drizzle.config.ts              # Drizzle ORM configuration
└── tsconfig.json                   # TypeScript configuration
```

## Core Source Directory (`src/`)

```plaintext
src/
├── app/                            # Next.js App Router routes
│   ├── (backend)/                  # Backend API routes
│   │   ├── api/                    # REST API endpoints
│   │   │   ├── auth/               # Authentication endpoints
│   │   │   └── webhooks/           # Webhook handlers for various auth providers
│   │   ├── middleware/             # Request middleware
│   │   ├── oidc/                   # OpenID Connect endpoints
│   │   ├── trpc/                   # tRPC API routes
│   │   │   ├── async/              # Async tRPC endpoints
│   │   │   ├── desktop/            # Desktop runtime endpoints
│   │   │   ├── edge/               # Edge runtime endpoints
│   │   │   ├── lambda/             # Lambda runtime endpoints
│   │   │   └── tools/              # Tools-specific endpoints
│   │   └── webapi/                 # Web API endpoints
│   │       ├── chat/               # Chat-related APIs for various providers
│   │       ├── models/             # Model management APIs
│   │       ├── plugin/             # Plugin system APIs
│   │       ├── stt/                # Speech-to-text APIs
│   │       ├── text-to-image/      # Image generation APIs
│   │       └── tts/                # Text-to-speech APIs
│   ├── [variants]/                 # Page route variants
│   │   ├── (main)/                 # Main application routes
│   │   │   ├── chat/               # Chat interface and workspace
│   │   │   ├── discover/           # Discover page (assistants, models, providers)
│   │   │   ├── files/              # File management interface
│   │   │   ├── image/              # Image generation interface
│   │   │   ├── profile/            # User profile and stats
│   │   │   ├── repos/              # Knowledge base repositories
│   │   │   └── settings/           # Application settings
│   │   └── @modal/                 # Modal routes
│   └── manifest.ts                 # PWA configuration
├── components/                     # Global shared components
│   ├── Analytics/                  # Analytics tracking components
│   ├── Error/                      # Error handling components
│   └── Loading/                    # Loading state components
├── config/                         # Application configuration
│   ├── featureFlags/               # Feature flags & experiments
│   └── modelProviders/             # Model provider configurations
├── features/                       # Feature components (UI Layer)
│   ├── AgentSetting/               # Agent configuration and management
│   ├── ChatInput/                  # Chat input with file upload and tools
│   ├── Conversation/               # Message display and interaction
│   ├── FileManager/                # File upload and knowledge base
│   └── PluginStore/                # Plugin marketplace and management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Xiedexiao/lobe-chat_rust](https://github.com/Xiedexiao/lobe-chat_rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
