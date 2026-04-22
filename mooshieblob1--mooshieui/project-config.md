---
trigger: always_on
description: MooshieUI is a desktop frontend for [ComfyUI](https://github.com/comfyanonymous/ComfyUI) built with **Svelte 5** + **Tauri v2** (Rust). It hides ComfyUI's node-graph complexity behind a clean UI for text-to-image, image-to-image, inpainting, and upscaling workflows.
---

# MooshieUI — Workspace Instructions

## Project Overview

MooshieUI is a desktop frontend for [ComfyUI](https://github.com/comfyanonymous/ComfyUI) built with **Svelte 5** + **Tauri v2** (Rust). It hides ComfyUI's node-graph complexity behind a clean UI for text-to-image, image-to-image, inpainting, and upscaling workflows.

## Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | Svelte 5, TypeScript 5, Tailwind CSS 4 |
| Desktop | Tauri v2 (Rust) |
| State | Svelte 5 runes (`$state`, `$derived`) — NOT Svelte stores |
| Persistence | `@tauri-apps/plugin-store` (JSON) |
| Backend API | ComfyUI REST + WebSocket (proxied via Rust) |
| Build | Vite 6 + `@sveltejs/vite-plugin-svelte` |

## Build & Run Commands

```bash
npm install              # Install frontend dependencies
npm run dev              # Vite dev server (port 1420)
npm run build            # Production frontend build → dist/
npm run tauri dev        # Full dev environment (Tauri + frontend hot-reload)
npm run tauri build      # Cross-platform binary build
cargo fmt                # Format Rust code (run inside src-tauri/)
cargo clippy             # Lint Rust code (run inside src-tauri/)
```

> **No test framework exists yet.** There are no test scripts, no vitest/jest, no Rust test modules.

## Architecture

```
Frontend (Svelte 5)  →  invoke()  →  Tauri Commands (Rust)  →  HTTP/WS  →  ComfyUI Server
                     ←  listen()  ←  Tauri Events (Rust)    ←  WebSocket ←
```

- **Frontend** (`src/`): UI only — generation params, progress display, gallery. No ComfyUI logic.
- **Tauri Backend** (`src-tauri/`): Command handlers, ComfyUI process management, WebSocket bridge, setup wizard.
- **Templates** (`src-tauri/src/templates/`): Build workflow JSON from generation params. Users never see node graphs.
- **Custom Nodes** (`comfyui-nodes/`): Python nodes installed into ComfyUI (tiled diffusion).

## Directory Structure

```
src/
├── main.ts                          # App entry point
├── App.svelte                       # Root component (routing, setup wizard, event listeners)
├── app.css                          # Tailwind imports + custom CSS variables
└── lib/
    ├── components/
    │   ├── generation/              # Generation controls (prompts, models, samplers, dimensions)
    │   ├── progress/                # Progress bar, live preview
    │   ├── gallery/                 # Image grid, lightbox
    │   └── setup/                   # First-run setup wizard
    ├── stores/                      # Svelte 5 rune-based state (*.svelte.ts)
    ├── types/                       # TypeScript interfaces (GenerationParams, OutputImage, etc.)
    └── utils/
        └── api.ts                   # Tauri invoke() wrappers

src-tauri/src/
├── main.rs                          # Windows subsystem entry, delegates to lib.rs
├── lib.rs                           # Tauri app builder, plugin registration, command registration
├── state.rs                         # AppState: RwLock<AppConfig>, Mutex<Process>, HTTP client
├── config.rs                        # AppConfig struct + JSON persistence
├── error.rs                         # AppError enum (thiserror)
├── setup.rs                         # One-click installer (uv, Python, ComfyUI, PyTorch)
├── commands/                        # Tauri command handlers
│   ├── api.rs                       # Model/sampler/history/gallery commands
│   ├── server.rs                    # start_comfyui, stop_comfyui, health checks
│   ├── websocket.rs                 # connect_ws, disconnect_ws
│   └── workflow.rs                  # generate (builds template → queues prompt)
├── comfyui/                         # ComfyUI integration layer
│   ├── client.rs                    # HTTP API wrapper (reqwest)
│   ├── process.rs                   # Subprocess lifecycle
│   ├── websocket.rs                 # WebSocket event handling → Tauri event emission
│   └── types.rs                     # Domain models
└── templates/                       # Workflow JSON builders
    ├── txt2img.rs
    ├── img2img.rs
    ├── inpainting.rs
    └── upscale.rs
```

## Coding Conventions

### Frontend (Svelte/TypeScript)

- **State management**: Use Svelte 5 runes (`$state`, `$derived`) in class-based singleton stores (`*.svelte.ts`). Do NOT use legacy Svelte stores (`writable`, `readable`).
- **Styling**: Tailwind CSS utility classes only. No `<style>` blocks in components. Dark mode is the only theme.
- **Color palette**: Neutral grays (`neutral-950` through `neutral-50`) + indigo accent. Custom CSS variable `--accent` defined in `app.css`.
- **Tauri IPC**: All `invoke()` calls go through `src/lib/utils/api.ts`. Don't call `invoke()` directly from components.
- **Tauri events**: Listen in `App.svelte` and update stores. Components read stores, not events.
- **Naming**: camelCase for TypeScript, kebab-case for CSS classes.
- **File naming**: `PascalCase.svelte` for components, `lowercase.svelte.ts` for stores, `lowercase.ts` for utilities.

### Backend (Rust)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mooshieblob1/MooshieUI](https://github.com/Mooshieblob1/MooshieUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
