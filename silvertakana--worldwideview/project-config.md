---
trigger: always_on
description: WorldWideView is a **real-time geospatial intelligence engine** that visualizes live global data on an interactive 3D globe. Built with **Next.js 16**, **CesiumJS**, **React 19**, and **Zustand**, it renders everything from live aircraft and maritime vessels to conflict events, satellites, and environmental data — all through a modular plugin architecture.
---

# WorldWideView — Agent Rules

## 1. Project Identity

WorldWideView is a **real-time geospatial intelligence engine** that visualizes live global data on an interactive 3D globe. Built with **Next.js 16**, **CesiumJS**, **React 19**, and **Zustand**, it renders everything from live aircraft and maritime vessels to conflict events, satellites, and environmental data — all through a modular plugin architecture.

### Target Inspiration
Our primary design, feature-set, and operational layout goal is to mimic the structure and capabilities of `www.worldmonitor.app`.
- **Reference Codebase**: [GitHub - koala73/worldmonitor](https://github.com/koala73/worldmonitor)
---

## 2. Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 16 (App Router, `output: "standalone"`) |
| Language | TypeScript 5, strict mode |
| 3D Engine | CesiumJS + Resium (Google Photorealistic 3D Tiles) |
| State | Zustand (slice-based: globe, layers, timeline, UI, filters, data, config, favorites, geojson) |
| Event Bus | Custom typed `DataBus` (pub/sub singleton) |
| Styling | Vanilla CSS — **no Tailwind** |
| Database | SQLite via Prisma (local), PostgreSQL (cloud) |
| Auth | NextAuth v5 beta (Credentials provider, JWT sessions) |
| Package Manager | pnpm (monorepo with `pnpm-workspace.yaml`) |
| Testing | Vitest + jsdom + React Testing Library |
| Deployment | Docker multi-stage build → Coolify |
| Analytics | Vercel Analytics / custom `trackEvent` |

---

## 3. Directory Structure

```
worldwideview/
├── src/
│   ├── app/               # Next.js App Router (pages, API routes, layouts)
│   │   ├── api/           # Server-side API routes (auth, aviation, camera, etc.)
│   │   ├── login/         # Login page
│   │   ├── setup/         # First-time setup page
│   │   └── globals.css    # Root stylesheet
│   ├── components/
│   │   ├── common/        # Shared UI: BootOverlay, FloatingWindow, PluginIcon
│   │   ├── layout/        # AppShell, Header, SearchBar, DataBusSubscriber
│   │   ├── panels/        # LayerPanel, EntityInfoCard, FilterPanel, GraphicsSettings
│   │   ├── timeline/      # Timeline component
│   │   ├── marketplace/   # Plugin install/unverified dialogs
│   │   ├── ui/            # Tooltip, ReloadToast
│   │   └── video/         # Floating video manager
│   ├── core/
│   │   ├── data/          # DataBus, PollingManager, CacheLayer, SmartFetcher
│   │   ├── filters/       # filterEngine (applies plugin filters to entities)
│   │   ├── globe/         # GlobeView, EntityRenderer, AnimationLoop, StackManager,
│   │   │   │                CameraController, InteractionHandler, SelectionHandler,
│   │   │   │                ModelManager, ImageryProviderFactory
│   │   │   └── hooks/     # useCameraActions, useEntityRendering, useModelRendering, etc.
│   │   ├── hooks/         # useBootSequence, useIsMobile, useMarketplaceSync
│   │   ├── plugins/       # PluginManager, PluginRegistry, PluginManifest,
│   │   │   │                loadPluginFromManifest, validateManifest, InstalledPluginsLoader
│   │   │   └── loaders/   # DeclarativePlugin, StaticDataPlugin, mapJsonToEntities
│   │   └── state/         # Zustand store + slices (config, data, globe, layers, timeline, ui, etc.)
│   ├── lib/               # auth, db, rateLimit, analytics, AIS stream, marketplace APIs
│   ├── plugins/           # GeoJSON plugin registrations
│   ├── styles/            # HUD animations CSS
│   ├── types/             # GeoJSON types, Umami types
│   └── generated/         # Prisma generated client (gitignored)
├── packages/              # pnpm monorepo workspace packages
│   ├── wwv-plugin-sdk/    # Plugin SDK: type definitions, manifest schema
│   ├── wwv-plugin-aviation/
│   ├── wwv-plugin-maritime/
│   ├── wwv-plugin-wildfire/
│   ├── wwv-plugin-borders/
│   ├── wwv-plugin-camera/
│   ├── wwv-plugin-military-aviation/
│   ├── wwv-plugin-satellite/
│   ├── wwv-plugin-iranwarlive/   # Includes standalone backend/ microservice
│   └── wwv-plugin-{airports,embassies,lighthouses,nuclear,seaports,spaceports,volcanoes}/
├── prisma/                # schema.prisma, migrations/
├── public/                # Static assets, Cesium workers, plugin GeoJSON data
├── scripts/               # Build scripts (copy-cesium, scaffold-osm-plugin, setup)
├── data/                  # SQLite database (gitignored, Docker volume)
├── Dockerfile             # Multi-stage production build
├── docker-compose.yml     # Main app + microservice backends
└── .agents/               # Agent documentation, rules, skills, workflows
```

---

## 4. Architecture Patterns

### 4.1 Plugin System (Core Abstraction)

Every data source is a **plugin** implementing the `WorldPlugin` interface from `@worldwideview/wwv-plugin-sdk`. The lifecycle utilizes a real-time WebSocket Firehose pipeline:

```text
PluginRegistry.register() → PluginManager.registerPlugin()
  → plugin.initialize(context)
  
Visibility Toggle → DataBusSubscriber subscribes to layer via WsClient

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [silvertakana/worldwideview](https://github.com/silvertakana/worldwideview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
