---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Minecraft Keybindings is a Next.js application for sharing and viewing Minecraft Java Edition keybinding configurations. Users authenticate via Discord OAuth, then manually set their Minecraft profile (MCID and UUID) and can create/edit their keybinding settings which are stored in a PostgreSQL database.

## Development Commands

### Setup & Installation
```bash
pnpm install
pnpm prisma generate
pnpm prisma db push
```

### Development
```bash
pnpm dev          # Start dev server at http://localhost:3000
pnpm build        # Build production bundle
pnpm start        # Start production server
pnpm lint         # Run ESLint
```

### Database Operations
```bash
pnpm prisma generate    # Generate Prisma Client after schema changes
pnpm prisma db push     # Push schema changes to database (development)
pnpm prisma studio      # Open Prisma Studio GUI
```

### Device Data Management
```bash
pnpm fetch-devices      # Fetch gaming mice and keyboards from Rakuten API
```

## Environment Variables

Required in `.env.local`:
- `DATABASE_URL` - PostgreSQL connection string (Neon recommended)
- `NEXTAUTH_URL` - Application URL (http://localhost:3000 for dev)
- `NEXTAUTH_SECRET` - Generated via `openssl rand -base64 32`
- `DISCORD_CLIENT_ID` - Discord app client ID
- `DISCORD_CLIENT_SECRET` - Discord app client secret
- `CRON_SECRET` - Secret for Vercel Cron job authentication (generated via `openssl rand -base64 32`)
- `RAKUTEN_APP_ID` - Rakuten API Application ID (optional, for device data fetching)

## Architecture

### Authentication Flow

The auth system ([lib/auth.ts](lib/auth.ts)) supports two authentication methods:

**1. MCID/Password Authentication (Credentials)**
- Users register with MCID, optional UUID, and password
- Password is hashed using bcryptjs (10 salt rounds)
- Direct login with MCID and password

**2. Discord OAuth (Optional)**
- User logs in with Discord account via NextAuth.js
- After first login, users are automatically redirected to MCID setup page
- Middleware ([middleware.ts](middleware.ts)) enforces MCID setup before accessing other pages

Note: Automatic Minecraft profile retrieval via Xbox Live API is not implemented due to Microsoft approval requirements.

### Data Model

The database schema follows NextAuth.js adapter requirements with additional Minecraft-specific fields:

**Account** (NextAuth model)
- OAuth provider accounts linked to users
- Stores provider tokens and metadata

**Session** (NextAuth model)
- User session tokens
- Links to User via userId

**User** (Prisma model in [prisma/schema.prisma](prisma/schema.prisma))
- Primary fields: `id`, `mcid` (username, unique), `uuid` (player ID, unique), `password` (hashed)
- Discord OAuth fields: `name`, `email`, `emailVerified`, `image` (NextAuth adapter requirements)
- Timestamps: `createdAt`, `updatedAt`
- Relationships: one-to-one with PlayerSettings, one-to-many with Account and Session

**PlayerSettings** (Prisma model)
- Stores complete keybinding configuration with defaults matching vanilla Minecraft
- Mouse settings: DPI, game sensitivity, Windows speed, acceleration, cm/180
- Keybindings: movement, actions, inventory, hotbar (stored as Minecraft key format: `key.keyboard.w`, `key.mouse.left`)
- JSON fields for advanced config:
  - `remappings` - Hardware-level key remaps (e.g., Caps Lock → Ctrl)
  - `externalTools` - AutoHotKey/macro configurations
  - `additionalSettings` - Future extensibility

### Key Format Convention

All keybindings use Minecraft's internal key naming:
- Keyboard: `key.keyboard.w`, `key.keyboard.left.shift`, `key.keyboard.space`
- Mouse: `key.mouse.left`, `key.mouse.right`, `key.mouse.middle`

This format is used throughout the schema defaults and should be preserved when adding new keybinding fields.

### API Routes

アプリケーションは17個のRESTful APIエンドポイントを提供しています。以下は主要なエンドポイントの概要です：

**プレイヤー情報取得（公開API）**
- `GET /api/player/[mcid]` - 個別プレイヤーの完全な情報を取得
- `GET /api/players` - 設定を持つ全プレイヤーのリストを取得

**認証・登録**
- `POST /api/auth/register` - 新規ユーザー登録（Mojang API連携）
- `POST /api/auth/login-or-register` - ログインまたは自動登録
- `GET /api/auth/check-mcid` - MCID存在チェック

**設定管理（要認証）**
- `GET /api/keybindings` - 認証済みユーザーの設定を取得
- `POST /api/keybindings` - プレイヤー設定を更新（管理者はゲストユーザーも編集可能）
- `DELETE /api/keybindings` - 全設定を削除

**アイテム配置**
- `GET /api/item-layouts` - アイテム配置を取得
- `POST /api/item-layouts` - アイテム配置を作成・更新
- `DELETE /api/item-layouts` - アイテム配置を削除

**ゲスト管理（管理者のみ）**
- `GET /api/guests` - ゲストユーザー一覧
- `POST /api/guests` - ゲストユーザー作成
- `DELETE /api/guests` - ゲストユーザー削除

**MCID同期**
- `GET /api/sync-mcid?uuid={uuid}` - 個別ユーザーのMCIDを同期
- `POST /api/sync-mcid` - 全ユーザーのMCIDを同期（Cron用）

**アバター**
- `GET /api/avatar?uuid={uuid}&size={size}` - Mojang APIからアバター画像を生成

### API Documentation

完全なAPI仕様書（17エンドポイント、リクエスト/レスポンス例、エラーハンドリング、使用例を含む）：
- **マークダウン形式**: [docs/API_SPECIFICATION.md](docs/API_SPECIFICATION.md)
- **OpenAPI形式**: [docs/openapi.yaml](docs/openapi.yaml)

### Session Management


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bafv4/minecraft-keybindings](https://github.com/bafv4/minecraft-keybindings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
