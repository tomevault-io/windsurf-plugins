---
trigger: always_on
description: This is a framework for creating Minecraft mods using Dart instead of Java. It uses a JNI bridge to connect Dart code running in a native library with Minecraft's Java runtime.
---

# Redstone Dart - Minecraft Modding with Dart

This is a framework for creating Minecraft mods using Dart instead of Java. It uses a JNI bridge to connect Dart code running in a native library with Minecraft's Java runtime.

## Project Structure

```
packages/
├── dart_mc/            # Main Dart API library for mod developers
├── java_mc_bridge/     # Java bridge code (server + client)
│   ├── src/main/       # Server-side/common Java code
│   └── src/client/     # Client-only Java code (renderers, GUI)
├── redstone_cli/       # CLI tool for building and running mods
# Note: JNI bridge code is in dart_mc/lib/src/jni/
├── native_mc_bridge/   # Native C++ JNI bridge library
├── redstone_test/      # Headless Minecraft E2E test framework
└── framework_tests/    # Comprehensive tests for the framework

example/
└── example_mod/        # Example mod demonstrating all features
```

## Architecture

### Client/Server Split

The codebase uses Fabric Loom's `splitEnvironmentSourceSets()` for client/server separation:

- **Server code** (`src/main/`): Uses `DartBridge` class via JNI
- **Client code** (`src/client/`): Uses `DartBridgeClient` class, marked with `@Environment(EnvType.CLIENT)`
- **Runtime checks**: Java callbacks use `!level.isClientSide()` to ensure server-only execution

### Bridge Communication

1. **Dart → Java**: Via `GenericJniBridge.callStaticVoidMethod()` with JNI signatures
2. **Java → Dart**: Via native callbacks registered during initialization
3. **Manifest system**: Dart writes to `.redstone/manifest.json`, CLI reads it for asset generation

## Key APIs

### Blocks & Items

```dart
class MyBlock extends CustomBlock {
  MyBlock() : super(
    id: 'mymod:my_block',
    settings: BlockSettings(hardness: 2.0),
    model: BlockModel.cubeAll(texture: 'assets/textures/block/my_block.png'),
  );
}

BlockRegistry.register(MyBlock());
```

### Entities with Models

Custom entities can have visual models rendered in-game:

```dart
class MyZombie extends CustomMonster {
  MyZombie() : super(
    id: 'mymod:my_zombie',
    settings: MonsterSettings(
      maxHealth: 30,
      attackDamage: 4,
      model: EntityModel.humanoid(
        texture: 'textures/entity/my_zombie.png',
      ),
    ),
  );
}

EntityRegistry.register(MyZombie());
```

**Available model types:**
- `EntityModel.humanoid(texture)` - Bipedal model (zombie/player-like)
- `EntityModel.quadruped(texture)` - Four-legged model (cow/pig-like)
- `EntityModel.simple(texture, scale)` - Basic scaled model
- `EntityModel.custom(texture)` - Custom model with texture only

Entities without a `model` field are invisible (use `NoopRenderer`).

### GUI Screens

```dart
class MyScreen extends Screen {
  @override
  void render(GuiGraphics graphics, int mouseX, int mouseY, double partialTick) {
    graphics.drawString('Hello World', 10, 10, color: 0xFFFFFF);
  }
}
```

## Development Commands

```bash
# In example/example_mod/
redstone run          # Build and run the mod with hot reload
redstone build        # Build without running
redstone generate     # Regenerate assets (blocks, items, textures)
redstone create       # Create a new mod project
redstone doctor       # Check system requirements
redstone devices      # List available devices
redstone upgrade      # Upgrade the CLI tool

# Testing entities in-game
/spawnzombie          # Spawn custom zombie
/spawncow             # Spawn custom cow
```

## Build System & Source Regeneration

### Automatic Rebuilds

The CLI automatically detects and rebuilds when sources change:

1. **Java Bridge Changes** (`packages/java_mc_bridge/src/`)
   - Auto-synced on `redstone run` or `redstone test`
   - Hash-based detection via `.redstone/version.json`

2. **Native C++ Bridge Changes** (`packages/native_mc_bridge/src/`)
   - Auto-rebuilt if CMake is available
   - Hash-based detection via `.redstone/version.json`

3. **Asset Generation** (blocks, items, entities)
   - Runs automatically on `redstone run`/`redstone test`
   - Manual: `redstone generate`

### Manual CMake Rebuild (Native Bridge)

When you need to manually rebuild the native C++ bridge:

```bash
cd packages/native_mc_bridge
cmake -B build .
cmake --build build --config release
```

**Output:** `dart_mc_bridge.dylib` (macOS), `dart_mc_bridge.dll` (Windows), `libdart_mc_bridge.so` (Linux)

**Requirements:** CMake 3.21+, C++17 compiler, JDK 21+

### Gradle Source Generation

To generate decompiled Minecraft sources for IDE navigation:

```bash
cd packages/java_mc_bridge
./gradlew genSources
```

## Navigating Minecraft Source Code

When implementing Java bridge code, you need to understand vanilla Minecraft's APIs and patterns. Decompiled Minecraft sources are available for reference.

### Source Location

```
packages/java_mc_bridge/mc-sources/
```

Contains 6,600+ decompiled Java files using official Mojang mappings.

### Setup Commands

```bash
# Full setup (generates and unpacks sources)
just java-setup

# Only unpack sources (if already generated)
just java-unpack-sources

# Clean Loom caches if having issues
just java-clean
```

### Common Source Paths

| Feature | Path |
|---------|------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Norbert515/redstone_dart](https://github.com/Norbert515/redstone_dart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
