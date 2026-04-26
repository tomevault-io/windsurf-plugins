---
trigger: always_on
description: `opencode-mobile` - Single mobile push notification plugin for OpenCode. Enables push notifications via Expo for mobile devices with tunnel management (ngrok/cloudflare/localtunnel). Built with TypeScript and Bun runtime.
---

# AGENTS.md

## Overview

`opencode-mobile` - Single mobile push notification plugin for OpenCode. Enables push notifications via Expo for mobile devices with tunnel management (ngrok/cloudflare/localtunnel). Built with TypeScript and Bun runtime.

## Build, Lint, and Test Commands

```bash
# Run the plugin
bun run index.ts

# Type-check only (no emit)
npm run typecheck
npx tsc --noEmit

# Compile TypeScript
npx tsc

# Build (type-check + compile)
npm run build

# Testing with vitest
npx vitest run                    # Run all tests
npx vitest run src/tunnel/        # Run tunnel tests
npx vitest run src/tunnel/localtunnel.test.ts  # Run specific test file
npx vitest run --reporter=verbose # Verbose output
npx vitest ui                     # Interactive UI (http://localhost:51204/__vitest__)

# Version and release
npm version patch && npm run build && npm publish  # Patch release
```

## Project Structure

```
plugin/
├── index.ts              # Entry point (plugin + HTTP server)
├── src/
│   ├── tunnel/           # Tunnel providers (ngrok, cloudflare, localtunnel)
│   │   ├── index.ts      # Unified interface, orchestrator
│   │   ├── localtunnel.ts# Localtunnel provider (62 lines, testable)
│   │   ├── cloudflare.ts # Cloudflare tunnel (117 lines, testable)
│   │   ├── ngrok.ts      # Ngrok with 4-strategy fallback (480 lines)
│   │   ├── qrcode.ts     # QR code utilities
│   │   ├── types.ts      # Type definitions
│   │   ├── *.test.ts     # Unit tests (vitest)
│   │   └── vitest.config.ts
│   ├── push/             # Push notification logic
│   │   ├── index.ts      # Barrel export
│   │   ├── types.ts      # Push types
│   │   ├── token-store.ts# Token persistence
│   │   ├── formatter.ts  # Notification formatting
│   │   ├── sender.ts     # Expo API sender
│   │   └── notification-handler.ts  # Session notification (commented)
│   └── proxy/            # Reverse proxy utilities
├── vitest.config.ts      # Test configuration
├── tsconfig.json         # TypeScript config (strict mode, bundler)
└── package.json          # Dependencies + scripts
```

## Code Style Guidelines

### Imports

```typescript
// Standard library - namespace imports
import * as fs from "fs";
import * as path from "path";

// External modules - named or default imports
import ngrok from "@ngrok/ngrok";
import qrcode from "qrcode";

// Types - use import type when only using types
import type { Plugin } from "@opencode-ai/plugin";
import type { TunnelConfig } from "./types";

// Group imports logically: types → external modules → internal modules
import type { Plugin } from "@opencode-ai/plugin";
import * as fs from "fs";
import * as path from "path";
import { startTunnel } from "./src/tunnel";
```

### Formatting

- **2 spaces** for indentation
- **Single quotes** for strings
ons** at end- **Semicol of statements
- **Trailing commas** in multi-line objects/arrays
- **Max line length**: ~100 characters (soft limit)

### Types

```typescript
// Interfaces for object shapes
interface PushToken {
  token: string;
  platform: "ios" | "android";
  deviceId: string;
  registeredAt: string;
}

// Type aliases for unions/primitives
type NotificationHandler = (notification: Notification) => Promise<void>;

// Explicit return types for public functions
function loadTokens(): PushToken[] {
  // ...
}

// Avoid `any` - use `unknown` with type guards when uncertain
function safeParse(data: unknown): Record<string, unknown> {
  if (typeof data === "string") {
    try {
      return JSON.parse(data);
    } catch {
      return {};
    }
  }
  return data as Record<string, unknown>;
}
```

### Naming Conventions

| Pattern | Convention | Example |
|---------|------------|---------|
| Constants | UPPER_SNAKE_CASE | `TOKEN_FILE`, `BUN_SERVER_PORT` |
| Functions/variables | camelCase | `loadTokens`, `startTunnel` |
| Interfaces/classes | PascalCase | `PushToken`, `TunnelConfig` |
| Private/internal | prefix `_` | `_bunServer`, `_pluginInitialized` |
| Booleans | prefix `is`, `has`, `should` | `isRunning`, `hasStarted` |

### Error Handling

```typescript
// Always wrap async operations in try-catch
try {
  await someAsyncOperation();
} catch (error: unknown) {
  // Log errors with module prefix
  console.error("[ModuleName] Error message:", error.message);
  
  // Provide context in error messages
  if (error.message?.includes("specific case")) {
    console.error("[PushPlugin] Handle specific error:", error.message);
  } else {
    console.error("[PushPlugin] Unexpected error:", error.message);
  }
}

// Handle specific error types when possible
if (error instanceof ValidationError) {
  // Handle validation errors
}
```

### Console Logging

- Use **module prefixes** in all console output: `[PushPlugin]`, `[Tunnel]`, `[Proxy]`
- Use **emojis** for status indicators: `✅`, `❌`, `💡`, `ℹ️`
- Log important steps and results

```typescript
console.log('[PushPlugin] Starting...');
console.error('[PushPlugin] Failed:', error.message);
console.log(`[Tunnel] URL: ${url}`);
console.log('✅ Server started successfully');
console.log('❌ Connection failed:', error.message);
```

### Async/Await


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doza62/opencode-mobile](https://github.com/doza62/opencode-mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
