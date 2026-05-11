---
trigger: always_on
description: This repository contains CloudDrop, a modern P2P file-sharing tool built on Cloudflare Workers and Durable Objects with a vanilla JS + CSS frontend.
---

# Agent Guide for CloudDrop

This repository contains CloudDrop, a modern P2P file-sharing tool built on Cloudflare Workers and Durable Objects with a vanilla JS + CSS frontend.
This guide summarizes the project, commands, coding standards, and architectural patterns for AI agents operating in this codebase.

## Project Overview

- **Runtime**: Cloudflare Workers + Durable Objects
- **Languages**: TypeScript (worker), vanilla JavaScript (frontend)
- **Core Component**: Durable Objects (`Room` class in `src/room.ts`)
- **Key Features**: WebSocket Hibernation API, WebRTC P2P transfer with relay fallback, end-to-end encryption
- **Deployment**: Wrangler
- **State**: Ephemeral signaling state + optional persistent room passwords
- **Frontend**: `public/` PWA (HTML/CSS/JS) with i18n and mobile-friendly UI

## Development Commands

### Build & Run

*   **Install Dependencies**
    ```bash
    npm install
    ```

*   **Start Local Development Server**
    ```bash
    npm run dev
    ```
    *Runs `wrangler dev`. This starts a local server (usually port 8787) emulating the Cloudflare Workers environment.*

*   **Deploy to Cloudflare**
    ```bash
    npm run deploy
    ```
    *Runs `wrangler deploy`. Requires `wrangler login` authentication.*

*   **Set TURN Secrets (Optional)**
    ```bash
    npx wrangler secret put TURN_KEY_ID
    npx wrangler secret put TURN_KEY_API_TOKEN
    ```
    *Enables Cloudflare TURN credentials for better NAT traversal.*

### Testing & Linting

*   **Run Tests**
    ```bash
    npm test
    ```
    *Current Status: No test suite is configured ("echo 'No tests yet'").*
    *Recommended: When adding tests, use `vitest` with `@cloudflare/vitest-pool-workers`.*

*   **Type Check**
    ```bash
    npx tsc --noEmit
    ```
    *Validates TypeScript types across the project without emitting output files.*

## Code Style & Guidelines

### TypeScript Configuration
- **Strict Mode**: `strict: true` is enabled in `tsconfig.json`. No implicit `any`.
- **Target**: `ES2022`.
- **Module Resolution**: `bundler`.
- **Types**:
    - Use `interface` for object shapes (e.g., `SignalingMessage`, `PeerAttachment`).
    - Explicitly type function arguments and return values (e.g., `: Promise<Response>`).
    - Use the `Env` interface for worker environment bindings (Durable Objects, Vars).

### Naming Conventions
- **Files**: kebab-case (e.g., `index.ts`, `room.ts`).
- **Classes**: PascalCase (e.g., `Room`).
- **Interfaces**: PascalCase (e.g., `PeerAttachment`, `Env`).
- **Functions & Variables**: camelCase (e.g., `handleWebSocket`, `roomCode`).
- **Constants**: UPPER_SNAKE_CASE (e.g., `WS_READY_STATE`).
- **Private Properties**: No underscore prefix required, just use the `private` keyword.

### Formatting
- **Indentation**: 2 spaces.
- **Semicolons**: Always use semicolons.
- **Strings**: Single quotes preferred, except for template literals.
- **Braces**: K&R style (opening brace on the same line).

### Error Handling
- **HTTP**: Return explicit `Response` objects with appropriate status codes.
    - `400 Bad Request`: Invalid input/parameters.
    - `404 Not Found`: Unknown route or resource.
    - `500 Internal Error`: Unexpected failures.
- **WebSockets**:
    - Wrap message handling in `try-catch` blocks to prevent Durable Object crashes.
    - Send typed error messages to the client: `{ type: 'error', error: 'CODE', message: '...' }`.
    - Log errors to the console (`console.error`), which streams to Cloudflare logs.

## Architecture & Patterns

### 1. Project Structure
- `src/index.ts`: **The Router**.
    - Handles incoming HTTP requests.
    - Generates Room IDs based on Client IP hashing (SHA-256).
    - Routes `/ws` requests to the specific `Room` Durable Object stub.
    - Handles static API endpoints (e.g., `/api/room-id`, `/api/ice-servers`).
- `src/room.ts`: **The State Machine**.
    - Implements the `DurableObject` interface.
    - Manages the WebSocket lifecycle.
    - Handles signaling logic (join, offer, answer, ice-candidate).
- `public/`: **The Client App**.
    - `index.html`, `style.css`, `manifest.json`, `js/` modules.
    - `js/webrtc.js` handles WebRTC + relay fallback + P2P recovery.
    - `js/crypto.js` handles encryption (AES-GCM + room password).
    - `js/i18n.js` handles translations (9 languages).
    - `js/app.js` bootstraps the app, wires UI to WebRTC + API, and manages UI state.
    - `js/ui.js` contains DOM helpers, view updates, toasts/modals, and device UI behaviors.
    - `js/config.js` centralizes constants (timeouts, chunk sizes, feature flags, API URLs).

### 2. Key HTTP Endpoints
- `GET /api/room-id`: returns a hashed room id derived from client IP.
- `GET /api/ice-servers`: returns STUN-only defaults or TURN credentials if configured.
- `POST /api/room/set-password?room=XXXXXX`: stores `passwordHash` in the room DO.
- `GET /api/room/check-password?room=XXXXXX`: returns whether room has a password.
- `GET /ws`: WebSocket upgrade, routed to a room DO based on room code.

### 3. Durable Objects & WebSocket Hibernation
This project uses the **WebSocket Hibernation API** for high performance and lower costs. Agents must follow these specific patterns:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DeH40/CloudDrop](https://github.com/DeH40/CloudDrop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
