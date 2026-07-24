---
trigger: always_on
description: > **For AI Assistants (Claude Code, Cursor, etc.)**
---

# CLAUDE.md - Tileserver RS Development Guide

> **For AI Assistants (Claude Code, Cursor, etc.)**
> This file is the single source of truth for tileserver-rs frontend + backend conventions. When this file conflicts with `.claude/skills/`, **this file wins**.

---

## Skills Integration & Priority

The frontend uses **vue-best-practices**, **nuxt-best-practices**, and **nuxt-seo-best-practices** skills from `.claude/skills/`. The backend uses **rust-skills** (179 rules). These provide generic guidelines.

**Priority rule: This CLAUDE.md ALWAYS takes precedence over generic skills when they conflict.**

### Known Conflicts (CLAUDE.md wins)

| Skill Says                                            | CLAUDE.md Says (Use This)                                                            |
| ----------------------------------------------------- | ------------------------------------------------------------------------------------ |
| Use `useFetch`/`useAsyncData` for data fetching       | Use **TanStack Query** with `$fetch` abstracted to `app/utils/api/` — see Rule #12   |
| Use Pinia for state management                        | Use **`useState`** for global state (SSR-safe, no provider wrapper needed)           |
| Use `provide/inject` for deep prop drilling           | Use **`useState`** for global state                                                  |
| Composable subfolder barrel exports (`auth/index.ts`) | **No barrels in auto-imported dirs** (`server/utils`, `shared/utils`) — see Rule #14 |
| `useQuery`/`useMutation` direct in components         | Wrap inside `app/utils/api/<feature>/*.{query,mutation}.ts` — see Rule #12           |

### What Skills Add (Not in CLAUDE.md)

- **Reactivity**: `ref` vs `reactive`, `toRefs`, `shallowRef`, `toRaw` for large data
- **Performance**: `v-once`, `v-memo`, `defineAsyncComponent`, `KeepAlive`
- **Templates**: `v-show` vs `v-if`, proper `:key` usage, avoid `v-if` + `v-for`
- **Composition API**: Single-responsibility composables, return refs not reactive objects

---

## Project Overview

**tileserver-rs** is a high-performance vector tile server built in Rust with a Nuxt 4 frontend. It serves vector tiles from PMTiles and MBTiles sources with native MapLibre rendering for raster tile generation.

### Key Capabilities
- PMTiles and MBTiles tile serving from local files
- HTTP-based PMTiles serving (remote files)
- **Native MapLibre GL rendering** via FFI bindings to MapLibre Native (C++)
- Raster tile generation (PNG/JPEG/WebP) from vector styles
- Static map image generation (like Mapbox Static API)
- TileJSON 3.0 metadata API
- MapLibre GL JS map viewer
- Style JSON and data inspector
- Configurable via TOML configuration
- **MLT (MapLibre Tiles) transcoding** — on-the-fly MLT↔MVT conversion (feature-gated)

---

## Tech Stack & Architecture

### Backend (Rust)
- **Axum 0.8** - Web framework
- **Tokio** - Async runtime
- **Tower-HTTP** - Middleware (CORS, compression, tracing)
- **Serde** - Serialization
- **Tracing** - Structured logging
- **Clap** - CLI argument parsing
- **mbgl-sys** - FFI bindings to MapLibre Native C++ for server-side rendering
- **mlt-core** - MLT tile parsing and decoding (optional, `mlt` feature)
- **prost** - Protobuf encoding for MVT tile generation (optional, `mlt` feature)
- **geo-types** - Geometry types used by mlt-core (optional, `mlt` feature)

### Frontend (Nuxt 4)
- **Nuxt 4** (v3.15) - Vue 3.5 framework with `app/` directory structure
- **Tailwind CSS v4** - Utility-first styling with `@tailwindcss/vite`
- **shadcn-vue** - UI components (configured at `app/components/ui/`)
- **MapLibre GL JS v4** - Map rendering
- **maplibre-gl-inspect** - Tile inspector
- **VueUse** - Vue composition utilities
- **TanStack AI Vue** - AI chat with `useChat` hook and `stream()` adapter
- **WebLLM** - Browser-local LLM inference via WebGPU (`@mlc-ai/web-llm`)
- **motion-v** - Vue animation library for UI transitions

### Infrastructure
- **Bun workspaces** - Monorepo package management
- **Docker** - Containerized deployment
- **Multi-stage builds** - Optimized image size

---

## ⛔ CRITICAL RULES - NEVER VIOLATE THESE

> **STOP AND READ BEFORE WRITING ANY CODE**
>
> These rules are **NON-NEGOTIABLE**. Violating them causes frustration and wasted time.

### 🚨 Rule #1: NEVER Define Types/Interfaces Inline in Vue Files

**NEVER define `interface` or `type` inside:**
- ❌ Vue components (`.vue` files)
- ❌ Composables (`composables/**/*.ts`)

**ALWAYS place types in the dedicated `app/types/` directory:**

```typescript
// ❌ WRONG - NEVER DO THIS
// app/pages/index.vue
interface StyleInfo {  // NO! Types don't belong in components!
  id: string;
  name: string;
}

// ❌ WRONG - NEVER DO THIS
// app/composables/useMapStyles.ts
export interface TileJSON { ... }  // NO! Types don't belong in composables!

// ✅ CORRECT - Types in dedicated files
// app/types/style.ts
export interface Style {
  id: string;
  name: string;
  url: string;
  version: number;
}

// Then import correctly:
import type { Style } from '~/types/style';
```

### 🚨 Rule #2: Use Existing Packages - Don't Reinvent

**Before writing custom code, CHECK if a package already exists:**

```typescript
// ❌ WRONG - Inline SVG strings when @nuxt/icon or lucide-vue-next exists

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vinayakkulkarni/tileserver-rs](https://github.com/vinayakkulkarni/tileserver-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
