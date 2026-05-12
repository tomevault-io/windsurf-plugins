---
trigger: always_on
description: Teak is a personal knowledge hub designed to help creative minds effortlessly collect, remember, and rediscover their most important ideas and inspirations.
---

Teak is a personal knowledge hub designed to help creative minds effortlessly collect, remember, and rediscover their most important ideas and inspirations.

## Quick Commands

```bash
# Install dependencies
bun install

# Dev (web + convex backend)
bun run dev

# Dev (all services)
bun run dev:all

# Individual services
bun run dev:web        # Next.js web + Convex
bun run dev:convex     # Convex backend only
bun run dev:api        # Hono API gateway
bun run dev:mobile     # Expo mobile app
bun run dev:desktop    # Electron desktop app
bun run dev:extension  # Browser extension + Convex
bun run dev:raycast    # Raycast extension
bun run dev:docs       # Documentation site

# Build/package extensions
bun run build:extension
bun run build:raycast
bun run publish:raycast

# Production build / start
bun run build
bun run start

# Lint & Typecheck
bun run lint
bun run typecheck

# Tests
bun run test

# Quality checks (Ultracite)
bun run check
bun run fix

# Pre-commit (same as git hook)
bun run pre-commit

# Clear caches
bun run clean

# Manage deps (in specific workspace)
bun add <package-name> --filter @teak/web
bun add --dev <package-name> --filter @teak/convex
```

```
teak/
├── apps/
│   ├── web/         # Next.js frontend (app router, shadcn/ui)
│   ├── api/         # Public API gateway (Hono + MCP)
│   ├── mobile/      # Expo RN mobile app
│   ├── desktop/     # Electron desktop app (React)
│   ├── extension/   # Chrome extension (Wxt)
│   ├── raycast/     # Raycast extension
│   └── docs/        # Documentation site (Astro + Starlight)
├── packages/
│   ├── convex/      # Convex backend (functions, workflows, schema, shared utils)
│   └── ui/          # Shared UI package (components, hooks, screens, feedback)
├── turbo.json       # Turborepo pipeline config
└── package.json     # Root package + workspaces
```

## Client-Server Patterns

- **Queries**: Real-time cached data via `convex-helpers/react/cache` `useQuery`, wrapped by `ConvexQueryCacheProvider`.
- **Mutations**: Server actions through `useMutation` / `useAction` from `@teak/convex`.
- **Auth context**: Better Auth sessions flow automatically to Convex with `@convex-dev/better-auth`.
- **App wrapping**: `ConvexClientProvider` + `ConvexQueryCacheProvider` wrap trees (web, mobile, extension) to share auth + cached queries; real-time updates propagate automatically.
- **Imports**: `import { api } from "@teak/convex"`, `import { Doc } from "@teak/convex/_generated/dataModel"`, `import { CARD_TYPES } from "@teak/convex/shared/constants"`.
- **Card Types**: text, link, image, video, audio, document, palette, quote.

## AI Processing Pipeline

- Orchestrated in `packages/convex/workflows/cardProcessing.ts` using `@convex-dev/workflow` with per-step retries.
- Sequence: classification (detect type + palette colors) → categorization (links; waits for metadata) → metadata (AI tags, summary, transcript) → renderables (media thumbnails; skips tiny originals; writes via internal mutations).
- Helpers: `packages/convex/workflows/functionRefs.ts` + `packages/convex/ai`.
- Link metadata: `packages/convex/workflows/linkMetadata.ts` via `startLinkMetadataWorkflow`;

## App Surfaces

- **Web (apps/web/)**: `src/app/(auth)/`, `src/app/(settings)/settings`, `src/globals.css`, `src/layout.tsx`, `src/page.tsx`; components include `ConvexClientProvider`, `SentryUserManager`, `JsonLd`, `GlobalFileDropProvider`; most UI components (card previews, grids, modals, forms, search, patterns) live in `@teak/ui`; Sentry error tracking via `instrumentation.ts`; config (`next.config.ts`, `eslint.config.mjs`, `components.json`).
- **Mobile (apps/mobile/)**: `app/(auth)/`, `app/(tabs)/(home)/|add/|settings/`, `_layout.tsx`; components (Expo UI, `CardItem`, `CardsGrid`, `CardPreviewSheet`, `ErrorBoundary`, `Logo`); `lib/hooks`, `lib/share`, `lib/auth-client.ts`, `lib/recording.ts`; `package.json`.
- **Desktop (apps/desktop/)**: Electron app with React frontend; `src/main/` for Electron main process; `src/preload/` for context bridge; `src/` for React renderer components; `src/pages/`, `src/hooks/`, `src/components/`, `src/lib/`; `forge.config.ts`, `vite.main.config.ts`, `vite.preload.config.ts`, `vite.renderer.config.ts`, `electron-builder.config.ts`.
- **Extension (apps/extension/)**: Wxt-based Chrome extension; `entrypoints/background.ts`, `entrypoints/content.ts`, `entrypoints/content/`, `entrypoints/popup/`; hooks (`useAutoSaveUrl`, `useContextMenuSave`, `useWebAppSession`); types (`contextMenu.ts`, `messages.ts`, `social.ts`); `utils/`, `lib/`, `scripts/`; `style.css`; assets in `public/`; `wxt.config.ts`; `package.json`; `tsconfig.json`.
- **Raycast (apps/raycast/)**: Raycast extension with commands (`quick-save`, `save-clipboard-url`, `save-current-browser-tab`, `search-cards`, `favorites`), AI tools (`search-cards`, `get-card`, `save-card`), API client helpers, and extension metadata/changelog.
- **API (apps/api/)**: Hono-based API gateway with REST routes (`src/routes/rest.ts`) and MCP routes (`src/routes/mcp.ts`); uses `@hono/mcp` + `@modelcontextprotocol/sdk`; source in `src/index.ts` and runtime entrypoint `src/server.ts`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [praveenjuge/teak](https://github.com/praveenjuge/teak) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
