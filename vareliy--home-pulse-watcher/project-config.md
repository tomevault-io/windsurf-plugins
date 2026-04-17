---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

HomePulse Watcher is a DIY power outage monitoring system. ESP32-C3/ESP32-C6 devices monitor power status and send HMAC-signed REST requests to this NestJS backend, which stores events and notifies users via Telegram.

**MVP Stage**: This project is in early development. No legacy concerns - database can be recreated from scratch as needed.

> **AI Assistant Shared Mental Model**: [`PROJECT_CONTEXT.md`](PROJECT_CONTEXT.md) — load this for a concise overview of architecture, domain rules (ADC sensing, HMAC/AES, debounce), incident history, and doc map.

## Build & Development Commands

```bash
# Start development server
npx nx serve api

# Build for production
npx nx build api

# Run unit tests
npx nx test api

# Run single test file
npx nx test api --testFile=app.service.spec.ts

# Run E2E tests
npx nx e2e api-e2e

# Lint
npx nx lint api

# Type check
npx nx typecheck api

# Build Docker image
npx nx docker:build api

# Database commands
npx prisma generate
npx prisma migrate dev --name <migration_name>
npx prisma migrate deploy  # production
```

## Architecture

Full architectural overview: [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)

### Core Layer Exports

```typescript
// Entities
(User, Device, UserDevice, PowerEvent);

// Enums
(PowerStatus((OFF = 0), (ON = 1)), DeviceRole(OWNER, EDITOR, VIEWER));

// Repository Interfaces
(IUserRepository, IDeviceRepository, IUserDeviceRepository, IPowerEventRepository);
```

### Infrastructure Layer (Pure TypeScript)

```typescript
// Repositories - plain classes, constructor injection of PrismaClient
const userRepo = new PrismaUserRepository(prismaClient);
const device = await userRepo.findByTelegramId(BigInt(123));

// Factory for PrismaClient singleton
(getPrismaClient(), disconnectPrisma());
```

### Telegram Bot Integration

- **Library**: Telegraf (TypeScript-native Telegram bot framework)
- **Module**: `apps/api/src/modules/telegram/`
- **Interaction model**: Button-driven; `/start` is the only slash command (user registration)
  - **Reply Keyboard**: Status, Devices, Settings, Help (persistent bottom menu)
  - **Inline Buttons**: Check Status, View History (in power event notifications)
  - **Settings**: Stateless inline keyboard menu for locale and timezone selection
- **Parse mode**: MarkdownV2 for all bot messages (`escape-markdown.ts` utility, `keyboard.builder.ts` for keyboard construction)
- **Notifications**: Event-driven via `@OnEvent(POWER_STATUS_CHANGED_EVENT)`
- **Authentication**: User verification via `telegramId` lookup before protected commands
- **Environment**: `TELEGRAM_BOT_TOKEN` (required), `TELEGRAM_ADMIN_CHAT_ID` (optional)
- **i18n**: `TranslationService` in `telegram/i18n/` — default locale `uk`, default timezone `Europe/Kyiv`. All user-facing strings are translated (Ukrainian + English). Date/time formatting uses per-user timezone via `Intl.DateTimeFormat`.

Bot follows the adapter pattern - handlers call existing Application Services, keeping business logic transport-agnostic.

### ESP32 Firmware

- **Location**: `firmware/` directory (ESP32-C3 and ESP32-C6 variants)
- **Build Tool**: PlatformIO with Arduino framework
- **Features**: WiFi, NTP time sync, HMAC-SHA256 signing, GPIO power detection
- **Configuration**: `config.h` (hardware), `secrets.h` (credentials - not tracked)
- **Documentation**: `docs/admin-guide.md`, `firmware/docs/FLASHING_GUIDE.md`

## Tech Stack

- **Monorepo**: Nx 22.4 with `@home-pulse-watcher/` prefix
- **Framework**: NestJS 11
- **Bundler**: Webpack via `@nx/webpack` (serverless bundling — all deps bundled except Prisma)
- **ORM**: Prisma 7.3 with PostgreSQL
- **Validation**: LIVR (custom rules use camelCase: `macAddress`, `hmacFormat`, `powerStatus`, `telegramId`)
- **Testing**: Jest 30 with SWC compiler
- **CLI**: nest-commander for admin tasks
- **Deployment**: Google Cloud Run (Docker multi-stage build)

## Database Models

- **User**: Telegram users (telegramId unique, locale default 'uk', timezone default 'Europe/Kyiv')
- **Device**: ESP32 devices (macAddress unique, encryptedSecret for HMAC verification)
- **UserDevice**: Many-to-many with role (VIEWER default)
- **PowerEvent**: Status changes (1=on, 0=off) with optional duration

## Build & Bundling

Webpack bundles **all** `node_modules` into `main.js` and `cli.js`, eliminating runtime dependency resolution issues. Only Prisma-related packages are kept external (they require native WASM binaries and generated code at runtime).

- **Config**: `apps/api/webpack.config.js`
- **External packages**: `@prisma/client`, `@prisma/adapter-pg`, `.prisma/client`, `pg`
- **Minimal package.json**: `apps/api/src/assets/package.json` — copied to `dist/`, lists only external deps for Docker `npm install`
- **NestJS lazy imports**: `@nestjs/microservices` and `@nestjs/websockets` are ignored via `IgnorePlugin` (optional peer deps, not installed)

## Coding Standards

- **Strict TypeScript**: No `any`, strict null checks
- **Composition over Inheritance**: Avoid deep class hierarchies
- **Interface-First**: Infrastructure adapters implement Core/Application interfaces

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vaReliy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
