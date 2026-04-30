---
trigger: always_on
description: Always use `pnpm` (not npm or yarn) for all commands. This is a **Turborepo monorepo**.
---

# DBDesk - Agent Guidelines

## Build/Test Commands

Always use `pnpm` (not npm or yarn) for all commands. This is a **Turborepo monorepo**.

### Root (all workspaces)
- **Build**: `pnpm run build` (turbo: builds all packages)
- **Dev**: `pnpm run dev` (turbo: starts all apps in dev)
- **Dev Desktop**: `pnpm run dev:desktop`
- **Dev Server**: `pnpm run dev:server`
- **Type Check**: `pnpm run typecheck`
- **Lint**: `pnpm run lint`
- **Format**: `pnpm run format`
- **Clean Reinstall**: `rm -rf node_modules apps/*/node_modules packages/*/node_modules .turbo apps/*/.turbo packages/*/.turbo && pnpm install`

### Desktop (`apps/desktop/`)
- **Build**: `pnpm run build` (typecheck + electron-vite build)
- **Dev**: `pnpm run dev` (electron-vite dev --noSandbox)
- **Type Check Node**: `pnpm run typecheck:node`
- **Type Check Web**: `pnpm run typecheck:web`

### Server (`apps/server/`)
- **Dev**: `pnpm run dev` (tsx watch src/index.ts)
- **Type Check**: `pnpm run typecheck`
- **DB Push**: `pnpm run db:push` (push Drizzle schema to database)
- **DB Migrate**: `pnpm run db:migrate`
- **DB Generate**: `pnpm run db:generate`
- **DB Studio**: `pnpm run db:studio`

### Server Setup (first time)
1. Copy `apps/server/.env.example` → `apps/server/.env`
2. Set `DATABASE_URL` to your PostgreSQL connection string
3. Generate `BETTER_AUTH_SECRET` with `openssl rand -base64 32`
4. Run `cd apps/server && pnpm run db:push` to create tables

## Code Style

- **Imports**: Use `@common/*`, `@renderer/*` path aliases; organize imports with prettier-plugin-organize-imports. Prefer `@renderer/` aliases for cross-feature imports; relative imports within the same feature module are fine.
- **Formatting**: Single quotes, no semicolons, 100 char width, no trailing commas (Prettier config)
- **Types**: Strict TypeScript, use `type` for object types, `interface` for extensible contracts
- **Naming**: PascalCase for components/classes, camelCase for functions/variables, kebab-case for files
- **Error Handling**: Use Result types, proper error boundaries in React components
- **React**: Use functional components, hooks, JSX runtime (no React imports needed)
- **UI Components**: Radix UI + Tailwind CSS, components in `apps/desktop/src/renderer/src/components/ui/`
- **State**: Zustand for global state, TanStack Query for server state, TanStack Form for forms

## Architecture

- **Monorepo**: Turborepo with pnpm workspaces
  - `apps/desktop/` — Electron desktop app (electron-vite 5, Electron 38)
  - `apps/server/` — Hono API server (@hono/node-server, port 3000)
  - `packages/shared/` — Shared types, Zod v4 schemas, IPC contract & utilities
  - `packages/api-client/` — Type-safe Hono RPC client (`hc<AppType>()`)
  - `packages/tsconfig/` — Shared TypeScript configs (base, node, react)
- Path aliases: `@common/*` → `src/common/*`, `@renderer/*` → `src/renderer/src/*`

### Desktop Main Process (`apps/desktop/src/main/`)

```
src/main/
├── index.ts                    # App entry, window creation, deep-link setup
├── connectionManager.ts        # Connection lifecycle management
├── storage.ts                  # Persistent storage
├── workspace-storage.ts        # Workspace state persistence
├── saved-queries-storage.ts    # Saved SQL queries persistence
├── adapters/                   # Database adapters (postgres, registry)
├── ipc/                        # Typed IPC handlers by domain
│   ├── index.ts                # registerAllIpcHandlers()
│   ├── typed-handle.ts         # typedHandle() with Zod validation
│   ├── adapter-handlers.ts
│   ├── auth-handlers.ts
│   ├── connection-handlers.ts
│   ├── query-handlers.ts
│   ├── saved-query-handlers.ts
│   ├── schema-handlers.ts
│   ├── table-handlers.ts
│   ├── update-handlers.ts
│   └── workspace-handlers.ts
├── lib/
│   ├── auth-manager.ts         # PKCE login flow, token refresh, session management
│   ├── auto-updater.ts         # electron-updater setup and event forwarding
│   ├── deep-link.ts            # dbdesk:// protocol handler, auth callback
│   ├── pkce.ts                 # PKCE SHA-256 challenge/verifier generation
│   ├── token-store.ts          # Electron safeStorage encrypted token persistence
│   ├── sql-parser.ts           # SQL parsing utilities
│   └── postgres/               # PostgreSQL-specific queries & utils
├── protocols/                  # Custom protocol handlers (asset server)
└── utils/                      # Error sanitization, validation helpers
```

### Desktop Preload (`apps/desktop/src/preload/`)

```
src/preload/
├── index.ts                    # contextBridge.exposeInMainWorld
├── typed-ipc.ts                # typedInvoke wrapper for IPC contract
├── dbdesk-api.ts               # Database operations API
├── window-api.ts               # Window control API
├── env-config.ts               # Build-time __API_URL__ / __WEB_URL__
└── index.d.ts                  # Type declarations for exposed APIs
```

### Desktop Renderer (`apps/desktop/src/renderer/src/`)

```
src/renderer/src/
├── main.tsx                          # Entry point
├── routeTree.gen.ts                  # TanStack Router generated
├── routes/                           # File-based routes
│   ├── __root.tsx
│   ├── index.tsx
│   ├── auth.tsx
│   └── $connectionId.tsx

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zexahq/dbdesk](https://github.com/zexahq/dbdesk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
