---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the **@sublay/node** package - the official Node.js SDK for Sublay. It's designed for server-side Node.js environments where React is not available or needed, such as server actions, backend APIs, scheduled jobs, webhooks, and CLI tools.

**Package Name**: @sublay/node
**Version**: 7.12.0
**Type**: Node.js SDK library (published to npm)

## Development Commands

```bash
# Build the package
pnpm build

# Whole-project type-check (tsc --noEmit) - catches errors in source files
# tsup's entry-graph-only build never reaches; emits nothing itself, so it
# can't clobber build's declaration output
pnpm build:types

# Build both (runs before publishing)
pnpm prepare

# Publishing is done from the monorepo root, not this package directory
# (run from /monorepo): pnpm node:publish-beta:patch / pnpm node:publish-prod:patch
# Use the :patch (or :minor) form — see "Publishing to npm" below for why.
```

## Core Architecture

### Module Structure

The SDK exposes **16 modules** bound on `SublayClient` (camelCase accessor in
parentheses). Every endpoint is reached with a **service key**; operations that
act on behalf of a user take an explicit `userId` (or `actingUserId` on the
nested `users` follow/connection routes and the `chat` target routes), since a
service key has no implicit session user.

```
src/
├── core/
│   └── client.ts           # HTTP client with axios instances
├── interfaces/             # TypeScript type definitions (Entity, Comment, User,
│                           #   Collection, Connection, Follow, Report, Space, …)
├── modules/
│   ├── entities/           # client.entities
│   ├── events/             # client.events
│   ├── comments/           # client.comments
│   ├── users/              # client.users (incl. nested follow/connection actions)
│   ├── spaces/             # client.spaces
│   ├── search/             # client.search
│   ├── auth/               # client.auth
│   ├── hosted-apps/        # client.hostedApps
│   ├── collections/        # client.collections      (service-key userId)
│   ├── connections/        # client.connections      (service-key userId)
│   ├── follows/            # client.follows          (service-key userId)
│   ├── reports/            # client.reports          (service-key userId)
│   ├── app-notifications/  # client.appNotifications (service-key userId)
│   ├── storage/            # client.storage          (service-key userId)
│   ├── push/               # client.push             (service-key userIds batch)
│   └── chat/               # client.chat             (service-key userId / actingUserId)
└── index.ts                # Main entry point with SublayClient class
```

> **Not bound (unmounted):** `oauth` only. It's a browser redirect flow with no
> meaningful server-to-server contract — the directory stays on disk but is not
> exposed on `SublayClient`.
>
> The space-scoped chat endpoints (`getSpaceConversation`, `moderateSpaceChatMessage`,
> `handleSpaceChatReport`) live on **`client.spaces`** (they're `/spaces/...` routes),
> not on `client.chat`.

### HTTP Client Configuration

The SDK uses three axios instances for different API endpoints:

- **projectInstance**: `https://api.sublay.io/v7/{projectId}` - Main project-scoped API
- **internalInstance**: `https://api.sublay.io/internal` - Internal operations (verification, admin)
- **baseInstance**: `https://api.sublay.io` - Base API endpoint

**Authentication Headers**:
- `Authorization: Bearer {apiKey}`
- `X-Sublay-Project-ID: {projectId}`
- `X-Sublay-Internal: true` (for internal operations)

### Initialization Pattern

```typescript
import { SublayClient } from '@sublay/node';

const client = await SublayClient.init({
    projectId: "your-project-id",
    apiKey: "your-api-key",
    isInternal?: boolean  // optional
});

// Automatically verifies credentials on init via /service/verify endpoint
```

## API Modules & Features

`SublayClient` binds **15 modules**. The source of truth is `src/index.ts` (the `bindModule` calls) and each module's `index.ts`. The full public surface and per-function props/returns are documented in `docs/v7/node-sdk/`.

**Acting on behalf of a user**: the SDK authenticates as the project (service key), not as an end user. So user-scoped functions take an explicit `userId` — the user the operation is performed as. A few routes act on one user *toward another* and take the actor as `actingUserId` while `userId` is the target: the nested follow/connection routes on the `users` module, and `chat.createDirectConversation` / `chat.addMember` / `chat.removeMember` / `chat.changeMemberRole`.

**Intentionally NOT bound**: `oauth` only (a browser redirect flow). The directory exists under `src/modules/` but is not exposed on `SublayClient`; do not document it.

### 1. Entities Module (16 functions)

Core content objects (posts, articles, products, listings, etc.).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [replyke/monorepo](https://github.com/replyke/monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
