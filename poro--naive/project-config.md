---
trigger: always_on
description: AI-native game engine — create worlds with YAML, Lua, and natural language.
---

# nAIVE Engine — Claude Code Context

## Project

AI-native game engine — create worlds with YAML, Lua, and natural language.

- **Language:** Rust (2021 edition)
- **Renderer:** wgpu + SLANG shaders
- **Physics:** Rapier3D
- **Scripting:** Lua 5.4 (mlua)
- **ECS:** hecs
- **Audio:** kira
- **Current version:** 0.1.18

## Workspace Crates

| Crate | Purpose |
|-------|---------|
| `naive-core` | Scene schema, shared types (YAML deserialization) |
| `naive-client` | Engine loop, renderer, physics, scripting, world management |
| `naive-runtime` | CLI binary (`naive`, `naive-runtime`, `naive_mcp`) |
| `naive-server` | Multiplayer server (stub — future) |
| `naive-web` | WASM + WebGPU build for browser playground (`crates/naive-web`, `cdylib`) |

## Key Source Files

| File | Responsibility |
|------|---------------|
| `crates/naive-core/src/scene.rs` | YAML scene schema types (`SceneDef`, `EntityDef`, `RigidBodyDef`, etc.) |
| `crates/naive-client/src/engine.rs` | Main `Engine` struct, game loop, camera, scene loading |
| `crates/naive-client/src/physics.rs` | `PhysicsWorld` — Rapier3D wrapper (bodies, colliders, CCD, impulse/force) |
| `crates/naive-client/src/world.rs` | `SceneWorld` (ECS + registry), `EntityCommandQueue`, entity spawning, pooling |
| `crates/naive-client/src/scripting.rs` | Lua API registration (physics, entity, camera, scene, events) |
| `crates/naive-client/src/renderer.rs` | wgpu render pipeline, instance buffers, particles |
| `crates/naive-client/src/pipeline/` | CompiledPipeline — YAML render-pass DAG (split into `def`/`resource`/`compiler`/`executor`/`mod`) |
| `crates/naive-client/src/debug_draw.rs` | Wireframe physics collider visualization (toggle: H key) |
| `crates/naive-client/src/command.rs` | MCP JSON-RPC command socket (TCP :6969) |
| `crates/naive-client/src/test_runner.rs` | Headless test runner for `naive test` CLI command |
| `crates/naive-client/src/dev_log.rs` | `naive submit-log` — POST dev.log as GitHub Issue |
| `crates/naive-client/src/demos.rs` | `naive demo` — 19 embedded demos with interactive browser |
| `crates/naive-client/src/editor_camera.rs` | Free fly camera for `naive edit` editor mode |
| `crates/naive-client/src/texture_cache.rs` | TextureCache — loads PNG/JPG/WEBP textures, creates wgpu bind groups |
| `crates/naive-client/src/beautify.rs` | Scene beautification pipeline (simple mesh → Gaussian splat conversion) |
| `crates/naive-web/src/lib.rs` | WASM entry point for browser playground (wgpu WebGPU init, panic hook, logging) |

## Architecture Patterns

- **Deferred commands:** Lua scripts push commands to `EntityCommandQueue`; engine processes them end-of-frame to avoid aliasing/dangling pointers.
- **Shared mutable state via `Rc<RefCell<T>>`:** Lua API closures capture `Rc<RefCell<PhysicsWorld>>` / `Rc<RefCell<SceneWorld>>` (and similar shared types — see `SharedSceneWorld`, `SharedPhysicsWorld`, etc. in `scripting.rs`) so mlua's `'static` closures can mutate engine state without `Arc<Mutex<>>` locking. Borrow discipline matters: always drop a `borrow_mut()` before dispatching events or calling user code that might re-enter.
- **In-place replacement for scene transitions:** `scene.load()` clears the ECS and re-initializes the shared `PhysicsWorld` via `borrow_mut()` so captured `Rc` handles remain valid across scene changes.
- **Pool manager:** Entity recycling for projectiles and spawned entities to avoid allocation churn.

## Tier Status

| Tier | Description | Status |
|------|-------------|--------|
| Tier 1 | Gameplay Primitives (health, damage, hitscan, projectiles, third-person camera) | **DONE v0.1.2** |
| Tier 2 | Production Foundations (dynamic instance buffer, entity lifecycle, pooling, particles, events) | **DONE v0.1.4** |
| Tier 2.5 | Physics & Scene API (impulse/force, velocity, CCD, collider materials, entity tags, camera shake, scene loading) | **DONE v0.1.7** |
| Tier 2.7 | DX & Code Quality (physics hot-reload, component patch coverage, pipeline modularization, HUD reload notifications, debug wireframes, kinematic bodies, convex decomposition) | **DONE v0.1.15** |
| Tier 3 | Visual & Interaction (texture mapping, procedural meshes, mouse picking, screen_to_ray) | **IN PROGRESS v0.1.18** |
| Tier 4 | GPU Scale (50K+ GPU compute entities, neighbor-grid collisions, flow field) | Planned |
| Tier 5 | Animation & Polish (skeletal animation, VAT, UI) | Planned |

## AI Asset Generation

Text-to-3D pipeline: prompt → FLUX.1 (2D) → Hunyuan3D (3D GLB) → engine.

### Environment Variables (`.env`)

| Variable | Purpose |
|----------|---------|
| `SLANG_DIR` | Path to vendored SLANG SDK (`vendor`) |
| `GATEWAY_URL` | Self-hosted GPU server URL (primary 3D gen backend) |
| `GATEWAY_KEY` | API key for GPU server |
| `HF_TOKEN` | HuggingFace API token (fallback 3D gen + 2D image gen) |
| `MODEL_SPACE` | HuggingFace Space for 3D generation |
| `MESHY_API_KEY` | Meshy AI API key (alternative 3D gen) |

**Never commit `.env`** — it contains secrets. Use `.env.example` as a template.

### MCP Servers (`.mcp.json`)

| Server | Tool | Purpose |
|--------|------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [poro/nAIVE](https://github.com/poro/nAIVE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
