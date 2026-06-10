---
trigger: always_on
description: Guide for agents working on the MCServerLauncher-Future-Web project.
---

# MCServerLauncher-Future-Web Guide

Guide for agents working on the MCServerLauncher-Future-Web project.

## Project Overview

**MCServerLauncher-Future-Web** is the web-based client suite for MCServerLauncher-Future. It's a monorepo containing a web panel and a Tauri-based desktop application, both sharing common UI components and logic.

- **Repository**: MCServerLauncher-Future-Web
- **Architecture**: Monorepo (Turborepo)
- **Package Manager**: Bun (migrated from pnpm in 2026)
- **License**: GPLv3
- **Team**: MCSLTeam
- **Copyright**: © 2022-2026 MCSLTeam
- **Codename**: CherryGrove
- **Version**: 0.1.0

## Essential Commands

```bash
# Install dependencies
bun install

# Web Panel Commands
bun run web:dev              # Start both frontend and backend dev servers
bun run web:dev:frontend     # Start only frontend dev server
bun run web:dev:backend      # Start only backend dev server (Rust)
bun run web:build            # Build web panel (frontend + backend)
bun run web:build:docker     # Build Docker image
bun run web:lint             # Lint web panel
bun run web:fix              # Fix linting issues

# Tauri App Commands
bun run app:dev              # Start Tauri app development
bun run app:build            # Build Tauri app
bun run app:lint             # Lint Tauri app
bun run app:fix              # Fix linting issues

# Global Commands
bun run build                # Build all packages
bun run lint                 # Lint all packages
bun run fix                  # Fix all linting issues
bun run test                 # Test all packages

# Package-specific Commands
bun run ui:lint              # Lint UI package
bun run shared:lint          # Lint shared package
```

## Project Structure

```
MCServerLauncher-Future-Web/
├── apps/
│   ├── web/                 # Web Panel (Vue 3 + Rust Actix-web)
│   │   ├── src/             # Rust backend source
│   │   │   ├── main.rs      # Entry point
│   │   │   ├── api.rs       # API routes
│   │   │   ├── user.rs      # User management
│   │   │   ├── token.rs     # Token/auth handling
│   │   │   ├── config.rs    # Configuration
│   │   │   └── utils.rs     # Utilities
│   │   ├── frontend/        # Vue 3 frontend (RsBuild)
│   │   ├── Cargo.toml       # Rust dependencies
│   │   ├── package.json     # Frontend dependencies
│   │   └── Dockerfile       # Docker build config
│   │
│   └── app/                 # Tauri Desktop App (Vue 3 + Tauri)
│       ├── src/             # Vue 3 frontend source
│       │   ├── App.vue      # Root component
│       │   ├── index.ts     # Entry point
│       │   └── components/  # App-specific components
│       ├── src-tauri/       # Tauri backend (Rust)
│       │   ├── src/         # Rust source
│       │   ├── Cargo.toml   # Rust dependencies
│       │   ├── tauri.conf.json        # Tauri config
│       │   └── tauri.macos.conf.json  # macOS-specific config
│       └── package.json     # Frontend dependencies
│
├── packages/
│   ├── configs/             # Shared configuration
│   │   ├── eslint/          # ESLint configs
│   │   ├── prettier/        # Prettier configs
│   │   └── ts/              # TypeScript configs
│   │
│   ├── locales/             # Internationalization (git submodule)
│   │   ├── locales/         # Translation files
│   │   ├── eula/            # EULA translations
│   │   └── scripts/         # Locale formatting scripts
│   │
│   ├── shared/              # Shared code and pages
│   │   └── src/
│   │       ├── assets/      # Shared assets
│   │       ├── components/  # Shared components
│   │       ├── layouts/     # Shared layouts
│   │       └── utils/       # Shared utilities
│   │
│   └── ui/                  # UI component library
│       └── src/             # UI components for MCSL Future
│
├── turbo.json               # Turborepo configuration
├── package.json             # Root package.json
├── bun.lock                 # Bun lockfile
├── Cargo.toml               # Rust workspace config
└── Cargo.lock               # Rust lockfile
```

## Technology Stack

### Frontend (Both Apps)

- **Framework**: Vue 3.5+ with Composition API
- **Language**: TypeScript 5.9+
- **Build Tool**: RsBuild 1.7+
- **State Management**: Pinia 3.0+
- **Routing**: Vue Router 5.0+
- **i18n**: Vue I18n 11.2+
- **HTTP Client**: Axios 1.13+
- **Utilities**: VueUse 14.2+, Day.js 1.11+
- **Validation**: Yup 1.7+

### Backend

**Web Panel (Rust)**:

- **Framework**: Actix-web 4
- **Edition**: Rust 2024
- **Logging**: env_logger 0.11+, log 0.4+
- **Serialization**: serde 1.0+, serde_json 1.0+
- **Utilities**: regex 1.12+, lazy_static 1.5+, tokio 1.49+
- **Security**: sha2 0.10+, hex 0.4+, rand 0.10+
- **Static Files**: include_dir 0.7+, mime_guess 2.0+

**Tauri App (Rust)**:

- **Framework**: Tauri 2.10+
- **Edition**: Rust 2024
- **Plugins**: notification, opener, os, process

### Build & Development

- **Monorepo**: Turborepo 2.8+
- **Package Manager**: Bun 1.3+ (migrated from pnpm)
- **Linting**: ESLint 10.0+, Prettier 3.8+
- **Type Checking**: vue-tsc 3.2+, TypeScript 6.0+

## Code Organization

### Web Panel (`apps/web`)

**Backend (Rust)**:

- `main.rs`: Actix-web server setup, static file serving, API routing
- `api.rs`: API endpoint handlers (~14KB, main business logic)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MCSLTeam/MCServerLauncher-Future-Web](https://github.com/MCSLTeam/MCServerLauncher-Future-Web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
