---
trigger: always_on
description: pnpm install        # Install deps + copy Cesium assets (postinstall)
---

# CLAUDE.md — Universe Web

## 基本规则

- **必须使用中文回复用户。**

## Development Commands

```bash
pnpm install        # Install deps + copy Cesium assets (postinstall)
pnpm dev            # Next.js dev server (localhost:3000)
pnpm build          # Production build
pnpm typecheck      # TypeScript strict check (no emit)
pnpm lint           # ESLint (flat config, next/core-web-vitals + next/typescript)
```

If Cesium assets are missing from `public/cesiumStatic/`, run `pnpm install` or `node scripts/copy-cesium-assets.mjs`. The postinstall script copies Workers/, ThirdParty/, Assets/, Widgets/ from the cesium package.

## Architecture

### Directory Layout

```
src/
├── agent/          # LLM tool-calling loop (OpenAI SDK)
├── app/            # Next.js App Router pages + API routes
├── components/     # React components (solar-system/, earth/)
├── server/         # Server-only logic (observation providers, geocode, satellites)
├── simulation/     # Astronomical math (Julian dates, solar geometry, ICRF)
├── store/          # Zustand state (viewerStore.ts — single global store)
└── types/          # Shared TypeScript types (agent, observation, global.d.ts)
```

### Client / Server Split

- **Client-only**: CesiumJS viewer, Zustand store, solar system experience. Cesium is dynamically imported with `ssr: false` to avoid server-side DOM errors.
- **Server-only**: API routes (`/api/agent`, `/api/query`, `/api/bodies`, `/api/geocode`, `/api/satellites`, `/api/providers`, `/api/layers`, `/api/analyze`). All use `runtime: 'nodejs'`.
- **Shared**: Type definitions, Zod tool schemas.

### CesiumJS Integration

- Viewer created in `SolarSystemCesiumScene.tsx` with no default UI (timeline, animation, base layer all disabled).
- Background color: `#020611`. Request render mode enabled.
- `CESIUM_BASE_URL` points to `/cesiumStatic`.
- Scene mode is always SCENE3D with no globe/sky.

### State Management

Single Zustand store (`useViewerStore`) holds: simulation time + playback state, selected body/location, inertial mode flag, layer visibility toggles, interface mode (explore/analysis), chat history, and ViewerController callbacks.

### Agent Tool-Calling Loop

`runAgent.ts`: OpenAI client → system prompt (built from AgentContext) → up to 4 tool-call iterations → returns reply + actions + citations + artifacts. 20 tools covering navigation, time control, rendering, observation queries, and UI control. Falls back to local pattern-matching (`fallback.ts`) when `OPENAI_API_KEY` is not set.

Actions are dispatched client-side via `applyClientActions.ts` which calls ViewerController methods and updates Zustand state.

## Key Conventions & Gotchas

### Scale Factors

- **Heliocentric (AU)**: `AU_TO_SCENE_METERS = 250_000_000` — 1 AU ≈ 250M scene meters.
- **Earth-Moon (km)**: `KM_TO_SCENE_METERS = 1_200` — used only in the `earthMoon` view preset.
- **Body radii**: `Math.max(900k, min(7.2M, log₁₀(radiusKm) × 1.1M))` for visual sizing. Selected bodies scale 1.2× for emphasis.

### ICRF (Inertial) Mode

When enabled, Cesium locks the camera to the inertial reference frame — Earth rotates beneath the camera instead of the camera orbiting Earth. Toggled via `set_inertial_mode` agent tool or store.

### OMM Normalization (Satellites)

CelesTrak OMM JSON fields arrive as mixed string/number types. The Zod schema accepts both and normalizes to strict types on parse.

### Cesium Client-Only Rendering

Never import Cesium at the top level of a server-rendered file. Use `next/dynamic` with `ssr: false` or guard with `typeof window !== 'undefined'`.

### View Presets

Four presets: `inner` (inner planets), `outer` (outer planets + moons), `full` (all), `earthMoon` (km-scale Earth-Moon detail). Each preset controls which bodies are rendered and the coordinate scale.

### Layer System

14 toggleable layers: `dayNight`, `atmosphere`, `cityMarkers`, `weatherClouds`, `weatherTemperature`, `moon`, `satellites`, `planetOrbits`, `planetLabels`, `majorMoons`, `spaceWeather`, `smallBodies`, `earthquakes`, `surfaceOverlays`.

## Environment Variables

```
OPENAI_API_KEY=           # Required for agent (falls back to pattern-matching without it)
OPENAI_MODEL=gpt-5        # Model selection
OPENAI_BASE_URL=           # Optional custom endpoint
GOOGLE_MAPS_API_KEY=       # Optional (future use)
NAIF_SPICE_KERNEL_ROOT=    # Optional (local SPICE kernel data)
```

## Key Files

| File | Purpose |
|------|---------|
| `src/store/viewerStore.ts` | Global Zustand state |
| `src/agent/runAgent.ts` | OpenAI tool-calling loop |
| `src/agent/toolSchemas.ts` | Zod tool schemas (20 tools) |
| `src/components/solar-system/SolarSystemCesiumScene.tsx` | Cesium viewer |
| `src/server/observation/solarSystem.ts` | Body catalog + JPL Horizons |
| `src/server/observation/query.ts` | Query router for all data providers |
| `src/simulation/astronomy.ts` | Earth state & solar geometry |
| `src/types/agent.ts` | Agent action/context/response types |
| `src/types/observation.ts` | Body descriptors, provider types |

## Design Context

### Users

面向**大众科普/教育**场景的地球观测站。用户是对太空、地球科学感兴趣的普通人——学生、科普爱好者、好奇访客。无需专业背景即可理解界面，通过 AI 对话和交互式地球探索来学习。

### Brand Personality

**探索 · 智能 · 优雅**

- **探索**: 每次交互都在揭示新发现，鼓励好奇心
- **智能**: AI 对话传递"懂你想问什么"的感觉，信息精确不堆砌
- **优雅**: 太空壮美不需花哨装饰，留白与动效克制考究

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zhangshuo1991) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
