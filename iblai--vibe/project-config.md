---
trigger: always_on
description: This file provides guidance for vibe coding with the ibl.ai platform.
---

# CLAUDE.md

This file provides guidance for vibe coding with the ibl.ai platform.

## MCP Server (Use First)

`.mcp.json` is configured with `@iblai/mcp`. **Always use MCP tools first** before searching the codebase:

```
get_component_info("ChatWidget")              # Props, usage, examples for any component
get_hook_info("useAdvancedChat")              # Hook parameters and return types
get_api_query_info("useGetUserMetadataQuery") # RTK Query endpoint details
get_provider_setup("auth")                    # Provider hierarchy and setup code
create_page_template("Dashboard", "mentor")   # Generate a page following ibl.ai patterns
get_playwright_helper_info("createConfig")    # E2E test utilities
```

## What is This

**Vibe** is a developer toolkit for shipping AI-powered apps fast on the ibl.ai platform. It provides:

- The `iblai` CLI for scaffolding apps and adding pre-built components
- Claude Code skills for guided, AI-assisted development
- The `@iblai/iblai-js` SDK with auth, chat, UI components, and data layer
- Connection to iblai.app -- a production backend with SSO auth, AI agents, analytics, and multi-tenancy

## Installing the CLI

```bash
iblai --version    # Check if already available
```

If not available, see the Prerequisites section in `/iblai-auth` for
installation options (npx or build from source).

## Getting Started

The CLI reads `PLATFORM` from `iblai.env` automatically. Pass `--platform`
only to override or when `iblai.env` is not set up.

> **AI Assistant:** When the user says "start a new project", "new app",
> "scaffold an app", or anything that creates a fresh codebase, **default
> to vibe-starter** (the option below). Only fall back to the vanilla
> Next.js or `iblai startapp agent` paths if the user explicitly declines
> the starter or asks for a minimal/custom setup.

### vibe-starter (recommended for new projects)

Pre-wired Next.js 16 + Tailwind v4 + shadcn/ui template with ibl.ai SSO
auth, a responsive navbar, and profile/account/notifications pages
already in place. Skips the manual `/iblai-auth`, `/iblai-navbar`,
`/iblai-profile`, `/iblai-account`, and `/iblai-notification` skills.

Clone into a temp directory and copy into the current directory before
installing (running `pnpm install` inside the cloned subdirectory causes
hardlink issues):

```bash
git clone -b spa https://github.com/iblai/vibe-starter.git vibe-starter-init
cp -a vibe-starter-init/. . && rm -rf vibe-starter-init
pnpm install
```

Then fill in `iblai.env` with `PLATFORM` and `TOKEN` and re-run
`iblai add auth` so the tenant key is written into `.env.local`.

### Vanilla Next.js + ibl.ai Features

```bash
npx create-next-app@latest iblai-init --yes
cp -a iblai-init/. . && rm -rf iblai-init
rm -rf node_modules && pnpm install
iblai add auth
iblai add chat
pnpm dev
```

### Full ibl.ai Agent App

Scaffold a complete app with auth, chat, and everything pre-configured.
Always create in a temp directory and copy back to the current directory:

```bash
iblai startapp agent -o iblai-init
cp -a iblai-init/<app-name>/. . && rm -rf iblai-init
rm -rf node_modules && pnpm install
cp .env.example .env.local
pnpm dev
```

### Add Features to Any Next.js App

```bash
iblai add auth           # SSO authentication
iblai add chat           # AI chat widget
```

Other features (profile, account, analytics, notifications, invitations,
workflows) are built using skills -- see `/iblai-profile`, `/iblai-account`,
`/iblai-analytics`, `/iblai-notification`, `/iblai-invite`, `/iblai-workflow`.

## Architecture

### Provider Chain

```
AuthProvider > TenantProvider > {children}
```

`initializeDataLayer` must be called with 5 arguments (data-layer v1.2+):

```typescript
initializeDataLayer(dmUrl, lmsUrl, legacyLmsUrl, storageService, httpErrorHandler)
```

### SDK Imports

```typescript
import { initializeDataLayer, mentorReducer } from "@iblai/iblai-js/data-layer";
import { AuthProvider, TenantProvider, useChatV2 } from "@iblai/iblai-js/web-utils";
import { Loader, TenantSwitch } from "@iblai/iblai-js/web-containers";
import { SsoLogin, UserProfileDropdown } from "@iblai/iblai-js/web-containers/next";
```

### Redux Store

`@reduxjs/toolkit` is deduplicated via webpack `resolve.alias` in `next.config.ts`. Without deduplication, SDK components use a different `ReactReduxContext` and RTK Query hooks silently return `undefined`.

## Environment

### `iblai.env` — Platform configuration

```bash
DOMAIN=iblai.app
PLATFORM=your-platform
TOKEN=your-api-token
VERCEL_TOKEN=your-vercel-token   # Optional — for mobile dev builds via Vercel
```

The CLI reads `DOMAIN`, `PLATFORM`, and `TOKEN` from `iblai.env` and derives
the `NEXT_PUBLIC_*` env vars into `.env.local` automatically.

> **Important:** `iblai.env` is NOT a replacement for `.env.local`. It only
> holds the 3 shorthand variables. Next.js reads its runtime env vars from
> `.env.local` / `.env` / `.env.development` as usual. The CLI bridges the
> two: it reads `iblai.env` and writes the derived `NEXT_PUBLIC_*` values
> into `.env.local`.

### `.env.local` — Next.js env vars (auto-derived)

```bash
NEXT_PUBLIC_API_BASE_URL=https://api.iblai.app
NEXT_PUBLIC_AUTH_URL=https://login.iblai.app
NEXT_PUBLIC_BASE_WS_URL=wss://asgi.data.iblai.app

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iblai/vibe](https://github.com/iblai/vibe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
