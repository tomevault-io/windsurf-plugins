---
trigger: always_on
description: Manages gateway credentials and auto-connect.
---

# AGENTS.md - Cove Development Guide

This file documents how Cove works for LLMs and developers working on the project.

## Project Overview

**Cove** is a WebUI for [OpenClaw](https://github.com/openclaw/openclaw) - an AI assistant gateway.

- **Stack**: Preact + Vite + TypeScript + Tailwind CSS
- **State**: Preact Signals (reactive, no Redux/Context complexity)
- **Styling**: Tailwind + CSS custom properties (themes)

## Architecture

```
src/
├── app.tsx              # Root component with router
├── main.tsx             # Entry point
├── lib/                 # Core libraries (non-component)
│   ├── gateway.ts       # WebSocket client for OpenClaw
│   ├── chat.ts          # Chat actions (sendMessage, loadHistory, etc.)
│   ├── session-utils.ts # Session key parsing helpers
│   ├── auth.ts          # Authentication state
│   ├── theme.ts         # Theme management
│   ├── i18n.ts          # Internationalization
│   ├── navigation.tsx   # Navigation config (single source of truth)
│   └── themes/          # Theme color definitions
├── hooks/               # Preact hooks
├── components/
│   ├── ui/              # Reusable primitives (Button, Input, Modal, etc.)
│   ├── chat/            # Chat-specific (MessageList, ChatInput, etc.)
│   ├── sessions/        # Session management (SessionItem, modals)
│   ├── layout/          # App shell (Sidebar, TopBar)
│   └── usage/           # Usage tracking components
├── views/               # Page-level components (ChatView, LoginView, etc.)
├── signals/             # Global state signals
├── types/               # TypeScript types
├── locales/             # Translation files
└── styles/              # CSS
```

## Key Systems

### 1. Gateway WebSocket Client (`src/lib/gateway.ts`)

Connects to OpenClaw gateway using its WebSocket protocol.

**Protocol Flow:**
1. Client opens WebSocket to gateway URL
2. Server sends `connect.challenge` event with `{ nonce, ts }`
3. Client sends `connect` request with:
   ```ts
   {
     type: "req",
     id: "req_1",
     method: "connect",
     params: {
       minProtocol: 3,
       maxProtocol: 3,
       client: {
         id: "webchat-ui",      // Required: known client ID
         displayName: "Cove",
         version: "0.1.0",
         platform: "macos",     // or windows/linux/ios/android/web
         mode: "webchat"        // Required: webchat/cli/ui/backend/node
       },
       auth: {
         token: "...",          // OR password, not both
       }
     }
   }
   ```
4. Server responds with `hello-ok` payload containing features, snapshot, policy

**Usage:**
```ts
import { connect, send, on, isConnected } from '@/lib/gateway'

// Connect
await connect({ url: 'wss://gateway.example.com', token: '...' })

// Send RPC request
const result = await send('session.list', { limit: 10 })

// Subscribe to events
on('chat.message', (payload) => console.log(payload))

// Check connection (reactive signal)
if (isConnected.value) { ... }
```

**Valid Client IDs** (from OpenClaw protocol):
- `webchat-ui` - Web chat interface (use this for Cove)
- `openclaw-control-ui` - Control panel
- `cli` - Command line
- `openclaw-macos/ios/android` - Native apps

### 2. Theme System (`src/lib/theme.ts`)

Multi-theme support with system preference detection.

**Built-in Themes:** light, dark, nord, dracula, solarized-light, solarized-dark

**How it works:**
- Themes define CSS custom properties (e.g., `--color-bg-primary`)
- `theme-script.ts` inlines in `<head>` to prevent FOUC
- Theme preference stored in localStorage
- System mode (light/dark) auto-switches themes

**Usage:**
```ts
import { setTheme, activeTheme, themePreference } from '@/lib/theme'

setTheme('dark')           // Set specific theme
setTheme('system')         // Use system preference
activeTheme.value.name     // Current theme name
```

### 3. i18n System (`src/lib/i18n.ts`)

Translation and locale-aware formatting.

**Usage:**
```ts
import { t, formatDate, formatRelativeTime } from '@/lib/i18n'

t('actions.send')                    // "Send"
t('messages.count', { count: 5 })    // "5 messages" (pluralized)
formatRelativeTime(date)             // "2 hours ago"
formatBytes(1048576)                 // "1 MB"
```

**Adding translations:**
1. Add keys to `src/locales/en.json`
2. Use dot notation: `t('section.subsection.key')`
3. Plurals: add `_plural` suffix key

### 4. Session Utilities (`src/lib/session-utils.ts`)

Shared helpers for working with session keys.

```ts
import { isMainSession, getAgentId, formatAgentName } from '@/lib/session-utils'

isMainSession('agent:main:main')     // true
getAgentId('agent:maude-pm:cron:uuid') // 'maude-pm'
formatAgentName('maude-pm')          // 'Maude PM'
```

### 5. UI Component Library (`src/components/ui/`)

Reusable primitives - **always check here before creating new components**:

- **Buttons**: `Button`, `IconButton`
- **Form**: `Input`, `Select`, `Toggle`, `Checkbox`, `FormField`
- **Layout**: `Card`, `Modal`, `ResizeHandle`
- **Feedback**: `Spinner`, `Badge`, `Toast`, `Skeleton`
- **Error**: `ErrorBoundary`, `InlineError`
- **Icons**: Re-exports from lucide-preact (see icons section)

```tsx
import { Button, Input, Modal, Toast } from '@/components/ui'
```

### 6. Auth State (`src/lib/auth.ts`)

Manages gateway credentials and auto-connect.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MaudeCode/cove](https://github.com/MaudeCode/cove) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
