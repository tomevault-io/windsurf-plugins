---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Root Operator** is a personal AI assistant for macOS powered by Claude Code channels. It combines chat, terminal access, identity-aware workspace bootstrapping, and secure remote access into a single desktop product.

## Build & Development Commands

```bash
# Install dependencies (triggers automatic native rebuild)
npm install

# Start app in development mode with HMR (recommended)
npm run dev:app

# Start app without HMR (loads built files)
npm start

# Rebuild native modules (node-pty, keytar) after node/electron version changes
npm run rebuild

# Build for macOS (unsigned, for local development)
npm run build:unsigned

# Build for macOS (requires Apple Developer credentials)
npm run build
```

### Development with HMR

The `npm run dev:app` command starts:
1. Vite dev server for renderer on port 5174 (tray app HMR)
2. Vite dev server for client on port 5175 (PWA client HMR)
3. Electron loading from the dev servers

Changes to React components are reflected immediately:
- `src/renderer/` - Tray app components (hot reload)
- `src/client/` - PWA client components (hot reload via proxy)

## Architecture Overview

### Process Architecture

**Main Process** (`main.js`):
- Electron main process that manages the app window, tray, and system integration
- HTTP server (port 22000) that serves PWA client and handles WebSocket connections
- Cloudflare tunnel manager that creates public URLs via `cloudflared` package
- PTY (pseudoterminal) manager using `node-pty` for shell process spawning
- E2E encryption layer using ECDH key exchange + AES-256-GCM
- Secure credential storage via `keytar` (macOS Keychain)

**Renderer Process** (`src/renderer/`):
- React + Tailwind + shadcn/ui components
- Entry point: `renderer.html` (built to `ui/dist/renderer.html`)
- Main app: `src/renderer/App.jsx`
- Components in `src/renderer/components/` (MainView, SettingsView, etc.)
- Communicates with main process via IPC (see `preload.js` for channel whitelist)

**Client/PWA** (`src/client/`):
- React + Tailwind + shadcn/ui (same stack as renderer)
- Entry point: `client.html` (built to `public/dist/client.html`)
- Main app: `src/client/App.jsx`
- Components: PairingScreen, Terminal, Header, VirtualKeyboard, EncryptionBadge
- Hooks: useWebSocket, useAuth, useE2E, useTerminal, useTerminalPersistence
- xterm.js terminal with fit and web-links addons
- E2E encryption with fingerprint verification (12-word BIP39)
- RSA-PSS authentication with 6-character pairing codes
- WebSocket auto-reconnection with exponential backoff
- Terminal content persistence via sessionStorage
- Can be added to iOS home screen as PWA

### Security Architecture

**IPC Security** (`preload.js`):
- Context isolation enabled
- Channel whitelist pattern - only specific IPC channels are allowed
- Renderer cannot access Node.js APIs directly

**ANSI Sanitization** (`main.js:42-101`):
- Filters dangerous ANSI escape sequences (OSC, DCS, APC, PM, SOS)
- Blocks clipboard manipulation (OSC 52) and title spoofing (OSC 0/1/2)
- Allows safe color palette sequences

**E2E Encryption Flow** (`main.js:103-285`, `public/client.js`):
1. Client connects via WebSocket
2. Server initiates ECDH key exchange, sends public key + salt
3. Client generates keypair, derives shared secret, sends public key
4. Both sides derive AES-256-GCM session key via HKDF
5. Both sides compute 12-word BIP39 fingerprint from shared secret + salt
6. User verifies fingerprint match between desktop tray menu and iOS client
7. All terminal I/O encrypted with AES-256-GCM (random IV per message)

**Authentication** (`main.js`, `src/client/hooks/useAuth.js`):
- RSA-PSS (2048-bit) public key authentication
- New devices: 6-character pairing code displayed on client, user enters on desktop to approve
- Returning devices: Server sends random challenge, client signs with private key, server verifies
- Challenge-response enforced for ALL reconnections (proves key possession, not just key ID)
- Approved keys stored in electron-store as `{kid, jwk}` pairs
- Rate limiting: max 5 auth attempts per connection
- Challenge expiry: 30 seconds

**Origin Validation**:
- WebSocket connections validated against Cloudflare tunnel URL
- Blocks connections from unauthorized origins

**WebSocket Reconnection** (`src/client/hooks/useWebSocket.js`):
- Automatic reconnection with exponential backoff (1s → 30s max)
- Jitter factor (±20%) to prevent thundering herd
- Heartbeat ping/pong every 25s to detect dead connections
- Handles network online/offline events
- Handles iOS PWA visibility changes (background/foreground)
- Server output buffer (1MB) preserves terminal history across reconnects

**Terminal Persistence** (`src/client/hooks/useTerminalPersistence.js`):
- Saves terminal content to sessionStorage (debounced 500ms)
- Restores on page reload before server buffer arrives
- Server buffer takes precedence as source of truth
- Auto-saves on page hide (iOS PWA backgrounding)
- Max 1MB content, uses sessionStorage for security (clears on tab close)

### State Management

**Electron Store**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hjertefolger/Root_Operator](https://github.com/hjertefolger/Root_Operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
