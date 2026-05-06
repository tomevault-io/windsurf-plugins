---
trigger: always_on
description: Instructions for AI coding assistants working on this project.
---

# AI Agents Guide

Instructions for AI coding assistants working on this project.

## Project Overview

Docker-based Hytale dedicated server with web admin panel. Two main components:

1. **Server Container**: Runs Hytale dedicated server (Java)
2. **Panel Container**: Node.js/TypeScript backend + Svelte 5 frontend

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Docker Host                             │
│  ┌─────────────────┐      ┌─────────────────────────────┐  │
│  │  hytale-server  │◄────►│       hytale-panel          │  │
│  │   (Java/Game)   │      │   (Node.js + Svelte 5)      │  │
│  │   Port: 5520    │      │   Ports: 3000, 5173         │  │
│  └─────────────────┘      └─────────────────────────────┘  │
│         ▲                            │                       │
│         │ /opt/hytale (volume)       │ docker.sock           │
│         └────────────────────────────┘                       │
└─────────────────────────────────────────────────────────────┘
```

## Project Structure

```
hytale-server/
├── Dockerfile              # Server container (Java)
├── entrypoint.sh           # Server startup script
├── docker-compose.yml      # Production
├── docker-compose.dev.yml  # Development with hot-reload
└── panel/
    ├── Dockerfile          # Panel production
    ├── Dockerfile.dev      # Panel development
    ├── tsconfig.base.json  # Shared TS config
    ├── biome.json          # Shared linter config
    ├── backend/
    │   ├── src/
    │   │   ├── config/     # Environment config
    │   │   ├── middleware/ # JWT auth
    │   │   ├── routes/     # REST API
    │   │   ├── services/   # Docker, files, mods, Modtale, CurseForge, updater
    │   │   ├── socket/     # Real-time handlers
    │   │   └── server.ts   # Entry point
    │   ├── __tests__/      # Jest tests (TypeScript)
    │   ├── jest.config.ts
    │   └── tsconfig.json
    └── frontend/
        ├── src/
        │   ├── lib/
        │   │   ├── components/  # Svelte 5 components
        │   │   ├── stores/      # Svelte stores
        │   │   ├── services/    # Socket client, API
        │   │   ├── types/       # TypeScript interfaces
        │   │   └── i18n/        # Locales (en, es, uk)
        │   └── main.ts
        ├── tsconfig.json
        └── vite.config.ts
```

## Key Files

### Backend (panel/backend/src/)

| File                     | Purpose                   |
| ------------------------ | ------------------------- |
| `server.ts`              | Express app entry         |
| `config/index.ts`        | Centralized configuration |
| `services/docker.ts`     | Docker API interactions   |
| `services/files.ts`      | File manager operations   |
| `services/mods.ts`       | Mod management            |
| `services/modtale.ts`    | Modtale API client        |
| `services/curseforge.ts` | CurseForge API client     |
| `services/updater.ts`    | Server update tracking    |
| `middleware/auth.ts`     | JWT authentication        |
| `socket/handlers.ts`     | WebSocket events          |

### Frontend (panel/frontend/src/lib/)

| Path                       | Purpose                                               |
| -------------------------- | ----------------------------------------------------- |
| `components/`              | Svelte 5 UI components                                |
| `stores/`                  | Application state (auth, server, files, mods, router) |
| `services/socketClient.ts` | Socket.IO wrapper                                     |
| `services/api.ts`          | REST API calls                                        |
| `types/index.ts`           | TypeScript interfaces                                 |
| `i18n/locales/`            | Translations (JSON)                                   |

## Tech Stack

### Backend

- **Node.js 25** (Alpine)
- **Express 5** + **Socket.IO 4**
- **TypeScript 5.9** (strict mode)
- **pnpm** package manager
- **Jest** + **ts-jest** for testing
- **Biome** for linting

### Frontend

- **Svelte 5** with runes
- **Vite 6** bundler
- **TypeScript** strict mode
- **svelte-i18n** for translations
- **Biome** + **Knip** for code quality

## Coding Patterns

### Backend (TypeScript)

```typescript
// Services return consistent objects
async function doSomething(): Promise<OperationResult> {
  try {
    // logic
    return { success: true, data };
  } catch (e) {
    return { success: false, error: (e as Error).message };
  }
}

// ESM imports with .js extension
import config from "./config/index.js";
import * as docker from "./services/docker.js";
```

### Frontend (Svelte 5 Runes)

```svelte
<script lang="ts">
  import { someStore } from '$lib/stores/example';

  // Props with $props()
  let { title, onClick }: { title: string; onClick: () => void } = $props();

  // Reactive state with $state()
  let count = $state(0);

  // Derived values with $derived()
  let doubled = $derived(count * 2);

  // Side effects with $effect()
  $effect(() => {
    console.log('Count changed:', count);
  });
</script>
```

### FORBIDDEN Svelte Patterns (deprecated)

```svelte
<!-- ❌ NEVER USE THESE -->
<script>
  export let prop;           // Use $props() instead
  $: derived = value * 2;    // Use $derived() instead

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ketbome/hytalepanel](https://github.com/Ketbome/hytalepanel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
