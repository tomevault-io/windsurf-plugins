---
trigger: always_on
description: Multi-loader Minecraft mod (Fabric + NeoForge + Paper) for 1.21.8 (Java 21). A GUI-based full-server song request system powered by Netease Cloud Music API.
---

# Music HUD - Agent Guide

Multi-loader Minecraft mod (Fabric + NeoForge + Paper) for 1.21.8 (Java 21). A GUI-based full-server song request system powered by Netease Cloud Music API.

## Build & Run

```bash
# Build specific loader (produces shadowed fat jars):
./gradlew fabric:build          # build/libs/music_hud-fabric-<version>.jar
./gradlew neoforge:build        # build/libs/music_hud-neoforge-<version>.jar
./gradlew paper:build           # build/libs/music_hud-paper-<version>.jar

# Run (Fabric/NeoForge via Loom):
./gradlew fabric:runClient
./gradlew neoforge:runClient
./gradlew fabric:runServer
./gradlew neoforge:runServer
```

## Critical Quirks

- **Tests are DISABLED by default** (`enabled = false` in `common/build.gradle:46`). To run: edit the file and change `enabled` to `true` — there is no Gradle property to override this at runtime.
- **No CI, no linter, no formatter, no typechecker** configured. Do not look for or run these.
- **Paper module is separate** — uses `paperweight.userdev` directly, NOT Architectury Loom. Paper is NOT in `settings.gradle` on this branch.
- **Build scripts are Groovy DSL** (`.gradle`), not Kotlin (`.gradle.kts`).
- **`core` module is a plain `java-library` (NO Loom)** — excluded by `build.gradle:16`: `configure(subprojects.findAll { it.name != 'core' })`.
  - Platform modules (`fabric`/`neoforge`) add `core` via a dedicated `coreLib` configuration that extends `compileClasspath` + `runtimeClasspath` but **NOT** `developmentFabric`/`developmentNeoForge` — this prevents Architectury Transformer from applying unnecessary transforms (`GenerateFakeFabricMod`, `RemapInjectables`).
- **`-Dfabric.dli.config` must be overridden in Fabric `loom.runs`** — Architectury Loom 1.17.487 generates DLI config at `.gradle/loom-cache/projects/<subproject>/launch.cfg` but the injector property points to `<subproject>/.gradle/loom-cache/launch.cfg`. Without the override, `dev-launch-injector` enters pass-through mode → mods and Minecraft assets won't load.
- **Mixin count: 6** — `SoundEngineMixin`, `GuiRendererHudMixin`, `ScreenMixin`, `SpanSetMixin`, `ReactiveMusicCompatMixin` + `MusicHudMixinPlugin` (plugin class).

## Architecture

```
core/            — platform-independent Java library (no Minecraft deps)
                   network codecs, payload interfaces, data beans, JMTC (SMTC/MPRIS),
                   server API interfaces, utility classes (RegistrationManager, etc.)
common/          — Architectury common module with Minecraft deps
                   UI (ModernUI), audio (stream decoders), mixins, platform service impls
fabric/          — Fabric loader adapter layer (shadow-jars common + core)
neoforge/        — NeoForge adapter layer (shadow-jars common + core)
paper/           — Paper/Bukkit plugin (on a separate branch, not in settings here)
```

`common` depends on `core` (`api project(':core')` in `common/build.gradle:24`). Platform modules shadow both `common` and `core` via `shadowBundle`.

The `configure(subprojects.findAll { it.name != 'core' })` block in `build.gradle:16` applies Loom to `common`, `fabric`, `neoforge` — but NOT `core`.

## Key Patterns

- **Service Locator** (not DI): `Environment.Platform.load()` uses `Class.forName()` + reflection to load platform-specific implementations. Interfaces: `ServerConfig`, `ClientConfig`, `IClientEventService`, `IServerEventService`, `INetworkRegister`, `IKeyRegistryService`.
- **Auto-Registration**: `RegistrationManager` loads classes by string array, instantiates `Register` implementors. `@RegisterMark` annotation marks registered classes. Called via `performCommonAutoRegistration()` / `performSideAutoRegistration()`.
- **Custom Network Protocol**: 30+ `CustomPacketPayload` classes in `network/payloads/`. Request/response cycle (`requestResponseCycle/`) + push messages (`pushMessages/`). Register via `INetworkRegister`.
- **Virtual Threads**: `MusicHud.EXECUTOR` = `Executors.newVirtualThreadPerTaskExecutor()`. Used for audio decoding, API server management, network I/O.
- **Mixin**: Config `music_hud.mixins.json`. 6 classes: `SoundEngineMixin`, `GuiRendererHudMixin`, `ScreenMixin`, `SpanSetMixin`, `ReactiveMusicCompatMixin` + `MusicHudMixinPlugin`.
- **Config**: Forge Config API Port (Fabric, shared with NeoForge) / native NeoForge `ModConfig` / Paper `config.yml`.

## Frameworks & Dependencies

- **ModernUI** (icyllis.modernui) — GUI framework, NOT vanilla MC widgets. Resolved from Gradle caches / Loom remap cache (may have flat JARs in `libs/` in some branches). See `ModernUI Library JARs` below for exact sources-JAR paths.
- **Lombok** heavily used — annotation processing is already configured.
- **JLayer** (MP3) + **JFLAC** (FLAC) — audio decoders, shaded into output jars.
- **Mojang mappings** — `loom.officialMojangMappings()`.

## ModernUI Library JARs

`idea_execute_tool read_file` can read files inside these sources JARs with `--file_path "<jar path>!/<entry>"`. `search_symbol` / `search_text` / `skill_search` do NOT index external libraries — never use them to locate classes inside these JARs; use the paths below directly.

### Path stability rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ephern/MusicHud](https://github.com/Ephern/MusicHud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
