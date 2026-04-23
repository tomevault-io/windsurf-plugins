---
trigger: always_on
description: electisSpace is an ESL (Electronic Shelf Label) management system for SoluM AIMS platform integration. It manages spaces (offices/rooms/chairs), people assignments, conference rooms, and label synchronization. The system supports web, Electron desktop, and Capacitor Android deployments from a single codebase.
---

# electisSpace — Project Instructions

## What This Project Is

electisSpace is an ESL (Electronic Shelf Label) management system for SoluM AIMS platform integration. It manages spaces (offices/rooms/chairs), people assignments, conference rooms, and label synchronization. The system supports web, Electron desktop, and Capacitor Android deployments from a single codebase.

**Owner:** Aviv Ben Waiss (AvivElectis)
**Repo:** `AvivElectis/electisSpace` (private)
**Current version:** Client v2.5.0 / Server v2.3.0

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Client | React 19, Vite (rolldown-vite), MUI 7, Zustand 5, React Hook Form + Zod, i18next, react-window |
| Server | Node.js 20, Express 4, Prisma 7, PostgreSQL, Redis + BullMQ, Socket.IO |
| Routing | HashRouter (`/#/` prefix on all URLs) |
| Auth | Access token in memory + refresh token as httpOnly cookie |
| i18n | English + Hebrew with RTL support |
| Desktop | Electron with auto-update via GitHub Releases |
| Mobile | Capacitor 7 (Android) |
| Testing | Vitest (unit), Playwright (E2E), MSW (mocking) |

## Architecture

### Client — Domain-Driven Design per Feature

```
src/features/<feature>/
  application/    ← Zustand stores, hooks, business logic orchestration
  domain/         ← TypeScript types, interfaces, enums, validation schemas
  infrastructure/ ← API calls (axios), adapters, external service integration
  presentation/   ← React components, pages, dialogs
  __tests__/      ← Unit tests (Vitest + Testing Library)
```

Shared code lives in `src/shared/` with the same DDD layers.

### Server — Feature Modules

```
server/src/features/<feature>/
  *.routes.ts     ← Express route definitions
  *.controller.ts ← Request handling, validation, response
  *.service.ts    ← Business logic
  *.types.ts      ← Zod schemas and TypeScript types
  __tests__/      ← Unit tests (Vitest)
```

### Key Features

| Feature | Client Path | Server Path |
|---------|-------------|-------------|
| Auth (login, 2FA, roles) | `src/features/auth/` | `server/src/features/auth/` |
| Dashboard | `src/features/dashboard/` | — |
| Spaces (offices/rooms/chairs) | `src/features/space/` | `server/src/features/spaces/` |
| People | `src/features/people/` | `server/src/features/people/` |
| Conference Rooms | `src/features/conference/` | `server/src/features/conference/` |
| Labels (ESL) | `src/features/labels/` | `server/src/features/labels/` |
| Settings | `src/features/settings/` | `server/src/features/settings/` |
| Sync (AIMS) | `src/features/sync/` | `server/src/features/sync/` |
| Import/Export | `src/features/import-export/` | — |
| Notifications | `src/features/notifications/` | — |
| Offline Mode | `src/features/offline-mode/` | — |

## Path Aliases

- `@features/*` → `src/features/*`
- `@shared/*` → `src/shared/*`
- `@test/*` → `src/test/*`

## Critical Patterns

### HashRouter — All URLs use `/#/` prefix
```typescript
// CORRECT
await page.goto('/#/spaces');
navigate('/spaces');

// WRONG — will 404
await page.goto('/spaces');
```

### Data Fetching — Pages use `useEffect` with guard dependencies
```typescript
useEffect(() => {
  if (isAppReady && activeStoreId) {
    fetchData();
  }
}, [isAppReady, activeStoreId]);
```

### MUI Form Fields — Use `getByLabel()` not `input[name="..."]`
MUI TextFields render without `name` attributes. Always locate inputs by label text.

### Spaces Table — Custom flexbox with react-window virtualization
The spaces table uses NO `<table>` or `[role="row"]` elements. Use the "Total ... - N" header text to verify data loaded.

### Auth Flow
1. Login → server returns access token (body) + refresh token (httpOnly cookie)
2. Access token stored in Zustand memory (never localStorage)
3. Axios interceptor auto-refreshes on 401 using refresh cookie
4. Auto-refresh debounced to prevent concurrent refresh calls

### SSE (Server-Sent Events)
- Endpoint: `GET /api/v1/stores/:storeId/events`
- Nginx requires `proxy_buffering off` and 24h read timeout
- Vite dev proxy strips Connection/Transfer-Encoding headers for HTTP/2

### Responsive Layout
- Desktop: Tab-based navigation (Dashboard/People/Conference Rooms/Labels)
- Mobile: Hamburger menu → MUI Drawer with list navigation
- Breakpoint detection: `Promise.race` between `[role="tablist"]` and `[aria-label="menu"]`

## Commands

### Development
```bash
npm run dev              # Start Vite dev server (port 3000)
cd server && npm run dev # Start Express dev server (port 3001)
```

### Testing
```bash
npm run test:unit        # Client unit tests (Vitest)
cd server && npx vitest run  # Server unit tests (Vitest)
npm run test:e2e         # E2E tests (Playwright, 4 workers)
npm run test:e2e:headed  # E2E with browser visible
```

### Building
```bash
npm run build            # Client production build (tsc + Vite)
cd server && npm run build  # Server TypeScript compilation
npm run electron:build   # Electron desktop installer
npm run android:build    # Capacitor Android build
```

### Database
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AvivElectis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
