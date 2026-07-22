---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ShieldBattery is a modern platform for playing StarCraft: Brood War/Remastered. It's a multi-language project:

| Directory    | Description                      | Stack                                       |
| ------------ | -------------------------------- | ------------------------------------------- |
| `client/`    | React web application            | TypeScript, Redux, Jotai, styled-components |
| `server/`    | Node.js backend server           | Koa.js, PostgreSQL, Redis, WebSockets       |
| `app/`       | Electron desktop application     | TypeScript, native OS integration           |
| `common/`    | Shared TypeScript code           | Types, utilities, IPC definitions           |
| `server-rs/` | Rust GraphQL server              | Axum, async-graphql, SQLx                   |
| `game/`      | Rust DLL injected into StarCraft | Windows API, egui                           |

**Architecture Rule**: `client/`, `server/`, and `app/` must not depend on each other. All can depend on `common/`.

## Quick Reference

| Task                | Pattern                                          | Location                                |
| ------------------- | ------------------------------------------------ | --------------------------------------- |
| Add HTTP endpoint   | `@httpApi` class + `@httpGet`/`@httpPost` method | `server/lib/<feature>/<feature>-api.ts` |
| Add WebSocket route | `@Mount` + `@Api` decorators                     | `server/lib/wsapi/`                     |
| Add Redux state     | `immerKeyedReducer` + actions                    | `client/<feature>/<feature>-reducer.ts` |
| Add local UI state  | Jotai atom                                       | `client/<feature>/<feature>-atoms.ts`   |
| Add shared type     | Interface in `common/`                           | Use typeshare if coming from Rust       |
| Add GraphQL query   | Resolver in server-rs + `pnpm gen-graphql`       | `server-rs/src/`                        |
| Test component      | Create devonly page                              | `client/<feature>/devonly/`             |

### Key File Locations

```
client/redux-hooks.ts                - useAppDispatch, useAppSelector (use instead of base hooks)
client/jotai-store.ts                - Global Jotai store instance
client/dispatch-registry.ts          - Global dispatch for non-React code
client/styles/colors.ts              - Theme CSS custom properties
client/material/                     - UI component library
client/gql/                          - Generated GraphQL types
common/urls.ts                       - urlPath, apiUrl tagged templates (auto-encode URLs)
common/ipc.ts                        - Electron IPC type definitions
server/lib/http/http-api.ts          - @httpApi decorator
server/lib/websockets/api-decorators.ts - WebSocket decorators
```

## Common Development Commands

```bash
pnpm run local-dev             # Run all dev services together (recommended)
pnpm run test                  # Unit tests (Vitest)
pnpm run test:integration      # Integration tests (Playwright)
pnpm run lint --fix            # ESLint + Prettier autofix
pnpm run typecheck             # TypeScript type checking

# Code generation (run after changing relevant source)
pnpm run gen-graphql           # Generate GraphQL types from schema
pnpm run gen-typeshare         # Generate Rust->TypeScript types
pnpm run gen-translations      # Generate translation files

# Database
pnpm run migrate:run           # Run migrations
pnpm run sqlx-prepare          # Update SQLx query metadata for Rust

# Rust game DLL
game\build.bat                 # Debug 32-bit
game\build.bat x86_64          # Debug 64-bit
```

## File Naming Conventions

| Pattern              | Purpose                  |
| -------------------- | ------------------------ |
| `*-reducer.ts`       | Redux reducer            |
| `*-atoms.ts`         | Jotai atoms              |
| `*-api.ts`           | HTTP API class           |
| `*-service.ts`       | Business logic service   |
| `socket-handlers.ts` | WebSocket event handlers |
| `devonly/`           | Development test pages   |

## Key Development Guidelines

### General

- Preserve `TODO(context)` and `NOTE(context)` comments unless completing the TODO
- **Comments must stand alone.** Don't write _deictic_ comments — ones whose meaning points at
  context outside the repo's current state: review findings ("Finding A3"), design docs, handoffs,
  tickets, chat threads, project phases, or before/after narrative ("previously...", "the new
  path", "today's behavior"). Those artifacts drift or get deleted, and the next reader wasn't
  there. The test: would this comment still be true and fully comprehensible after every document,
  conversation, and branch around the change is gone? If not, rewrite it to state the invariant,
  hazard, or constraint in the code's own terms. Provenance and review trail belong in commit
  messages, where such references are welcome. (`TODO(context)`/`NOTE(context)` tags are the one
  sanctioned forward pointer.)
- Delete unused code during refactoring

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShieldBattery/ShieldBattery](https://github.com/ShieldBattery/ShieldBattery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
