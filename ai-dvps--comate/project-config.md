---
trigger: always_on
description: Comate is a desktop AI workspace that wraps Claude Code in a native Electron app. It uses a hybrid architecture: a **React 18 + Vite** frontend, an **Express.js** sidecar server, and an **Electron** desktop shell.
---

# Comate — Claude Code Project Guide

Comate is a desktop AI workspace that wraps Claude Code in a native Electron app. It uses a hybrid architecture: a **React 18 + Vite** frontend, an **Express.js** sidecar server, and an **Electron** desktop shell.

## Quick Commands

| Command | Purpose |
|---------|---------|
| `npm run dev:server` | Start the Express backend with hot reload |
| `npm run dev:client` | Start the Vite dev server (port 5173) |
| `npm run dev:electron` | Start the Electron desktop app (Vite dev client + shell) |
| `npm run lint` | Run ESLint on `.ts`/`.tsx` |
| `npm run typecheck` | Type-check client, server, and Electron sources |
| `npm test` | Run server, client, Electron, and build-script unit tests |
| `npm run check` | Run the complete lint, type-check, and unit-test suite |
| `npm run test:client` | Run jsdom-based component/hook tests |
| `npm run test:server` | Run `node:test` server tests (excludes `src/server/vendor/`) |
| `npm run test:browser` | Run Playwright browser tests |
| `npm run release` | Build sidecar + CDP gate + Electron production bundle (electron-builder) |

> Do **not** run `npm run dev` alongside `npm run dev:electron` — both start Vite and will conflict on port 5173.

## Architecture

```
┌─────────────────┐     WebSocket / HTTP      ┌──────────────────┐
│  Electron shell │  ←──────────────────────→  │  Express server  │
│  (electron/)    │                           │  (src/server/)   │
└────────┬────────┘                           └────────┬─────────┘
         │                                             │
         │  Vite dev client / bundled UI               │  sidecar Node process
         ↓                                             ↓
┌─────────────────┐                          ┌──────────────────┐
│  React UI       │                          │  SQLite, Claude  │
│  (src/client/)  │                          │  SDK, file I/O   │
└─────────────────┘                          └──────────────────┘
```

- **Frontend** (`src/client/`): React 18, Zustand stores, Tailwind CSS, Radix primitives, `lucide-react` icons.
- **Backend** (`src/server/`): Express API routes, service layer, SQLite storage via `better-sqlite3`.
- **Desktop shell** (`electron/`): Electron main process + preload, sidecar Node binary lifecycle, native browser views (WebContentsView), tray, updater.
- **Plugins** (`claude-code-plugin/`): Built-in local plugin marketplace shipped with the app bundle.
- **WeCom CLI** (`packages/wecom-cli/`): Workspace-packaged oclif-style CLI for WeChat Work integration.

## Project Conventions

### TypeScript & Module Rules

- Target: ES2020, module: ESNext, moduleResolution: bundler.
- Strict mode is on, including `noUnusedLocals` and `noUnusedParameters`.
- Import paths use `.js` extensions for compiled server files (e.g., `./routes/workspaces.js`), even though source is TypeScript. Vite handles client imports without extensions.
- Path aliases:
  - `@/` → `src/client/`
  - `@server/` → `src/server/`

### Code Style

- ESLint with `@typescript-eslint/recommended` and `react-hooks/recommended`.
- React Refresh rule enabled; prefer named component exports unless constant-export patterns are needed.
- Use `const` arrow functions for handlers; prefer functional `setState` updates when depending on previous state.
- Tailwind classes are composed with `cn()` from `src/client/components/ui/utils.ts`.

### File Naming

- Components: PascalCase (`ChatPanel.tsx`, `SessionListItem.tsx`).
- Stores/hooks/utils: camelCase (`workspace-store.ts`, `use-theme.ts`).
- Server routes/models/services: kebab-case (`workspace-commands.ts`, `sqlite-store.ts`).
- Tests: co-located as `<name>.test.ts` or `<name>.browser.test.tsx`.

## Client Patterns

### State Management

- Use **Zustand** stores in `src/client/stores/`. Keep store logic close to the feature domain (e.g., `chat-store.ts`, `workspace-store.ts`).
- Select only the slices a component needs to avoid unnecessary re-renders.
- Stores talk to the Express backend via `fetch` to `/api/*` routes.

### Components

- Reusable UI primitives live in `src/client/components/ui/`.
- Feature components live directly under `src/client/components/`.
- Tool-specific renderers live in `src/client/components/tool-renderers/`.
- Use `useTranslation('namespace')` for all user-facing strings; namespaces are in `src/client/i18n/{en,zh-CN}/`.

### Theming

- Dark mode is class-based (`dark` class on root). Tailwind config uses CSS variables (`--color-bg`, `--color-surface`, etc.).
- Theme utilities are in `src/client/hooks/use-theme.ts`.

## Server Patterns

### Routes

- Routes are Express `Router` instances in `src/server/routes/`.
- Return JSON shapes like `{ workspaces }`, `{ workspace }`, `{ error }` for consistency.
- Validate required fields inline; return `400` for bad input, `404` when not found, `500` for unexpected errors.

### Services

- Business logic and long-lived state live in `src/server/services/`.
- Services are generally imported as singletons (e.g., `chatService`, `wecomBotService`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ai-dvps/comate](https://github.com/ai-dvps/comate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
