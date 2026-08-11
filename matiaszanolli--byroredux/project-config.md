---
trigger: always_on
description: Clean rebuild of the Gamebryo/Creation engine lineage in Rust + C++ with Vulkan.
---

# ByroRedux

Clean rebuild of the Gamebryo/Creation engine lineage in Rust + C++ with Vulkan.
Long-term goal: load and render content from Gamebryo/Creation-era games.

## Quick Reference

```bash
cargo check                    # Type check (fast)
cargo test -p byroredux-core    # Run ECS/core tests (162 tests)
cargo test                     # Full workspace tests
cargo run                      # Launch engine (spinning cube demo)
cargo build --release          # Release build
```

### Debug CLI
```bash
cargo run -p byro-dbg                       # Connect to running engine (port 9876)
BYRO_DEBUG_PORT=8080 cargo run -p byro-dbg  # Custom port
```

### Smoke tests
Manual end-to-end checks that need a Vulkan device + on-disk game data
(out of `cargo test` scope). All follow the same `--bench-hold` →
`byro-dbg`-attach pattern documented in
[`docs/smoke-tests/README.md`](docs/smoke-tests/README.md). Currently:
[`docs/smoke-tests/m41-equip.sh`](docs/smoke-tests/m41-equip.sh)
verifies Skyrim+ / FO4 NPC outfit equip end-to-end.

### Shader Compilation
```bash
cd crates/renderer/shaders
glslangValidator -V triangle.vert -o triangle.vert.spv
glslangValidator -V triangle.frag -o triangle.frag.spv
```

## Workspace Structure

```
byroredux/              Binary — game loop, scene setup, systems
  src/main.rs              App struct, ApplicationHandler (winit event loop), main()
  src/components.rs        Marker components (Spinning, AlphaBlend, TwoSided, Decal) + app resources
  src/systems.rs           ECS systems: fly camera, animation, transform propagation, spin, stats
  src/scene.rs             Scene setup, NIF loading (load_nif_bytes, load_nif_from_args)
  src/asset_provider/      BSA/BA2-backed texture and mesh extraction
    mod.rs                   TextureProvider, resolve_texture, re-exports
    archive.rs               GameArchive — wraps BSA (Oblivion-Skyrim SE) or BA2 (FO4-Starfield)
    texture.rs                File-data lookup by searching BSA/BA2 archives
    material.rs              Material-path resolution incl. Starfield materialsbeta.cdb
    script.rs                Compiled Papyrus (.pex) lookup by script name (M47.2 attach path)
    tests.rs                  Archive-provider regression tests
  src/render/              Per-frame render data collection (build_render_data), split by pass
    mod.rs                   Top-level build_render_data + shared collection state
    camera.rs                View-projection + frustum setup
    lights.rs                Light collection
    particles.rs             Particle billboard emission
    sky.rs                   Sky parameter assembly
    skinned.rs               Skinned-mesh palette pass
    static_meshes.rs         Static mesh main loop
    water.rs                 Water-plane re-emit
    *_tests.rs               Per-pass regression tests (bone palette overflow, draw sort key, frustum, …)
  src/anim_convert.rs      NIF→core animation clip conversion, subtree name map
  src/commands/             Console commands (help, stats, entities, systems), split by topic
    mod.rs                   Command dispatch table
    scene.rs                 Scene / lighting / material / script-state commands
    assets.rs                Texture / mesh / skin diagnostic commands
    actor_value.rs           setav/modav — live-edit an actor's ActorValues
    condition.rs             cond — evaluate a CTDA condition function live
    world_info.rs            Engine / world / memory introspection commands
    view.rs                  Camera + selection/picking commands
    shared.rs                Cross-command formatting helpers + shared import prelude
  src/helpers.rs            add_child, world_resource_set utilities
  src/cell_loader.rs        ESM cell loading (interior + exterior)
crates/
  core/                      ECS, math (glam), types, string interning, form IDs
    src/ecs/                 World, Component, Storage, Query, System, Scheduler, Resource
    src/ecs/components/      Transform, GlobalTransform, Parent, Children, Camera, MeshHandle, Name,
                             FormIdComponent, LightSource, AnimatedVisibility/Alpha/Color
    src/ecs/resources/       DeltaTime, TotalTime, EngineConfig
      mod.rs                   Built-in engine resources
      skin_slot_pool.rs        Per-entity persistent bone-palette slot pool (bind_inverses SSBO, M29.6)
    src/animation/           Animation engine
      types.rs               CycleType, KeyType, key structs, channels, AnimationClip
      registry.rs            AnimationClipRegistry (Resource)
      player.rs              AnimationPlayer (Component), advance_time()
      stack.rs               AnimationLayer, AnimationStack, advance_stack(), sample_blended_transform()
      root_motion.rs         RootMotionDelta, split_root_motion()
      interpolation.rs       find_key_pair, hermite, TBC tangents, sample_translation/rotation/scale/float/color/bool
      text_events.rs         collect_text_key_events()
    src/form_id.rs           FormId, PluginId, LocalFormId, FormIdPair, FormIdPool
    src/string/              StringPool, FixedString
  plugin/                    Plugin system — manifests, records, DataStore, conflict resolution

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matiaszanolli/ByroRedux](https://github.com/matiaszanolli/ByroRedux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
