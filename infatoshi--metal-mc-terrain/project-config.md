---
trigger: always_on
description: Identify and fix client-side rendering bottlenecks in SkyFactory One (Minecraft 1.16.5 + Forge 36.2.34). The server is already running and not our problem. We care about FPS, frame time spikes, and rendering throughput on the client.
---

# SkyFactory One Client-Side Performance Mod

## Goal

Identify and fix client-side rendering bottlenecks in SkyFactory One (Minecraft 1.16.5 + Forge 36.2.34). The server is already running and not our problem. We care about FPS, frame time spikes, and rendering throughput on the client.

## Build & Deploy

```bash
# One-liner: build mod jar and copy to SkyFactory mods folder
./build.sh

# Or manually:
JAVA_HOME=/opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home ./gradlew build --no-daemon
cp build/libs/modid-1.0.jar ~/Documents/curseforge/minecraft/Instances/SkyFactory\ One/mods/patched-overlay-1.0.jar
```

Launch from CurseForge (handles Microsoft auth). CLI launch script exists at `./launch.sh` but only works offline.

## Java Versions

- **Gradle build**: Java 17 (`/opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home`)
- **Game runtime**: Java 8 (`~/Documents/curseforge/minecraft/Install/java/jre-legacy/Contents/Home/bin/java`)
- **build.gradle toolchain**: targets Java 8

## File Locations

```
~/skyfactory-dev/                              # This project (Forge MDK workspace)
  src/main/java/com/example/examplemod/        # Mod source
    ExampleMod.java                            # Entry point, join message handler
    PatchedOverlay.java                        # HUD overlay (red "[PATCHED]" text)
  src/main/resources/META-INF/mods.toml        # Mod metadata
  build.sh                                     # Build + deploy script
  launch.sh                                    # CLI launch (offline only)

~/Documents/curseforge/minecraft/
  Install/
    versions/1.16.5/1.16.5.jar                 # Vanilla client jar (obfuscated)
    versions/forge-36.2.34/forge-36.2.34.jar   # Forge jar
    libraries/org/lwjgl/                       # LWJGL 3.2.1 (OpenGL bindings)
    java/jre-legacy/                           # Bundled Java 8
    assets/                                    # Game assets
    natives/                                   # Native libraries
  Instances/SkyFactory One/
    mods/                                      # ~80+ mod jars, our jar goes here
    config/                                    # Mod configs

/tmp/mc-decompile/
  client_mappings.txt                          # Mojang official mappings for 1.16.5
  decompiled/                                  # CFR-decompiled classes (obfuscated names)
    eae.java                                   # LevelRenderer
    ecu.java                                   # ChunkRenderDispatcher
    dzz.java                                   # GameRenderer
    com/mojang/blaze3d/systems/RenderSystem.java
```

## Mappings (obfuscated -> real names)

The vanilla jar uses obfuscated class names. Mojang mappings at `/tmp/mc-decompile/client_mappings.txt`.

| Real Name | Obfuscated | Role |
|-----------|-----------|------|
| `net.minecraft.client.renderer.LevelRenderer` | `eae` | Main world renderer (chunks, entities, block entities) |
| `net.minecraft.client.renderer.chunk.ChunkRenderDispatcher` | `ecu` | Chunk compilation thread pool + GPU upload |
| `net.minecraft.client.renderer.GameRenderer` | `dzz` | Top-level render loop |
| `com.mojang.blaze3d.systems.RenderSystem` | (not obfuscated) | OpenGL wrapper |
| `net.minecraft.client.renderer.entity.EntityRenderDispatcher` | `eet` | Entity rendering dispatch |
| `net.minecraft.client.renderer.entity.EntityRenderer` | `eeu` | Base entity renderer |
| `net.minecraft.client.renderer.chunk.ChunkRenderDispatcher$RenderChunk` | `ecu$c` | Single chunk section (16x16x16) |
| `net.minecraft.client.renderer.chunk.ChunkRenderDispatcher$CompiledChunk` | `ecu$b` | Compiled chunk data |
| `net.minecraft.client.renderer.chunk.VisGraph` | `ecw` | Occlusion/visibility graph per chunk |
| `net.minecraft.client.renderer.chunk.RenderChunkRegion` | `ecv` | 3x3 chunk snapshot for compilation |

ForgeGradle uses `official` mappings channel. **Method/field names** use Mojang's official names (e.g. `renderLevel`, `chunkRenderDispatcher`). **Class names** use MCP/SRG intermediary names, NOT Mojang names:

| Mojang Class Name | Dev Class Name (use this in code) |
|---|---|
| `LevelRenderer` | `WorldRenderer` (`net.minecraft.client.renderer.WorldRenderer`) |
| `PoseStack` | `MatrixStack` (`com.mojang.blaze3d.matrix.MatrixStack`) |
| `Camera` | `ActiveRenderInfo` (`net.minecraft.client.renderer.ActiveRenderInfo`) |
| `Matrix4f` | `Matrix4f` (`net.minecraft.util.math.vector.Matrix4f`) |
| `ProfilerFiller` | `IProfiler` (`net.minecraft.profiler.IProfiler`) |
| `KeyMapping` | `KeyBinding` (`net.minecraft.client.settings.KeyBinding`) |
| `ChunkRenderDispatcher` | `ChunkRenderDispatcher` (same) |
| `GameRenderer` | `GameRenderer` (same) |
| `LightTexture` | `LightTexture` (same) |
| `TileEntity` | `TileEntity` (`net.minecraft.tileentity.TileEntity`) |

## Graphics API

**OpenGL** via LWJGL 3.2.1. NOT Metal, NOT Vulkan.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Infatoshi/metal-mc-terrain](https://github.com/Infatoshi/metal-mc-terrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
