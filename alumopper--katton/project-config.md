---
trigger: always_on
description: Minecraft Fabric, NeoForge mod & Paper plugin for Kotlin scripting with hot reload. Target MC version 26.1.2.
---

# Katton

Minecraft Fabric, NeoForge mod & Paper plugin for Kotlin scripting with hot reload. Target MC version 26.1.2.

## Build & Run

- **Java 25** required (enforced by `project.options.release = 25` and `JVM target = 25`).
- **Gradle 9.3.0** (wrapper checked in at `gradle/wrapper/gradle-wrapper.properties`).
- Run via Gradle:
  - `./gradlew :fabric:runClient` — Fabric client
  - `./gradlew :neoforge:runClient` — NeoForge client
  - `./gradlew :paper:runServer` — Paper server (via `xyz.jpenilla.run-paper`)
- Build: `./gradlew build` (produces jars for each submodule).
  - Paper uses **Shadow** (`com.gradleup.shadow`) to produce a fat jar with embedded Kotlin runtime.
- Configuration cache disabled (`org.gradle.configuration-cache=false` in `gradle.properties`) — IntelliJ + Fabric Loom incompatibility.
- **No tests** exist anywhere in the repo (a `test` sourceSet exists in root `build.gradle` pointing to `Katton-Example/`, but no test framework is declared and no actual test files exist).

## Project Structure

Multi-module Gradle build (Groovy DSL, Kotlin plugins via `org.jetbrains.kotlin.jvm` 2.3.10):

| Module | Build system | Entrypoint |
|---|---|---|
| `:common` | Fabric Loom (access widener) + Kotlin JVM | `top.katton.Katton.java` — platform-agnostic init utility |
| `:fabric` | Fabric Loom | `top.katton.KattonFabric.java` — `ModInitializer` |
| | | `top.katton.KattonClientFabric.java` — `ClientModInitializer` |
| `:neoforge` | NeoForge ModDev (`net.neoforged.moddev` 2.0.141) | `top.katton.KattonNeoForge.java` — `@Mod` |
| | | `top.katton.KattonClientNeoForge.java` — `@EventBusSubscriber` (client) |
| `:paper` | Paperweight userdev (`io.papermc.paperweight.userdev` 2.0.0-beta.21) + Shadow | `top.katton.paper.KattonPaperPlugin.java` — `JavaPlugin` |
| `:buildSrc` | Custom Kotlin plugin | `ApiDocGeneratorPlugin` — generates VitePress API docs |

- Common logic (script engine, networking, registry, API) lives in `common/src/main/kotlin/top/katton/` and `common/src/main/java/top/katton/`.
- Platform-specific event APIs mirror each other: **14 event classes** each under `fabric/`, `neoforge/`, and `paper/` (see [Event System](#event-system)).
- Mixins: Fabric mixins in Java under `fabric/src/main/java/top/katton/mixin/` (**16 files**), NeoForge mixins under `neoforge/src/main/java/top/katton/mixin/` (**30 files**). Paper has **no mixins** — it operates as a standard Paper plugin. A `common/src/main/resources/kts4mc.mixins.json` exists with an empty mixin list.
- Kotlin scripting libraries are embedded (`include` in Fabric, `jarJar` in NeoForge, `implementation`→shadow in Paper, `compileOnly` in common).
- NeoForge uses **access transformers** (`src/main/resources/META-INF/accesstransformer.cfg`) in addition to the access widener from common. Paper uses **no access transformers**.
- A **stale** `common/src/main/resources/fabric.mod.json` exists referencing non-existent entrypoint classes (`Katton` and `KattonClient` as Fabric entrypoints) — ignore it; only the fabric-module copy is active.

### Common Source Package Map

```
common/src/main/kotlin/top/katton/
├── api/          │ Public script API (annotations, registry functions, inject, render, mod, DP callers, datapacks)
│   ├── inject/     │ UnsafeApi.kt — before/after/replace/redirect hooks (ByteBuddy)
│   ├── registry/   │ Per-type registry helpers (Item, Block, Entity, etc.)
│   ├── mod/        │ Item/Block modification API (KubeJS-like)
│   ├── dpcaller/   │ 13 DP caller wrappers (BlockApi, EntityApi, ItemApi, etc.)
│   ├── datapack/   │ Recipes.kt, Tags.kt
│   ├── event/      │ KattonEventsArg.kt — shared event argument data classes
│   └── event/managed/ │ ManagedEvents.kt — cross-platform managed listener abstraction
├── bridge/       │ KattonBridge.kt — inter-module bridge
├── bridger/      │ Entity/LootTable/Enchantment bridges + EventResult/ModifyContext
├── client/       │ ScriptPackUi.kt, ReloadProgressOverlay.kt, ReloadProgressState
├── command/      │ ScriptCommand.kt — `/katton` command tree
├── datapack/     │ ServerDatapackManager.kt — reloadable server datapack injection
├── engine/       │ ScriptEngine.kt, ScriptEnvironment.kt, InjectionManager.kt, JavaCompilationUtil.kt
├── network/      │ ServerNetworking.kt, 3 packet types (request/hashlist/bundle)
├── pack/         │ ScriptPackManager.kt, ScriptPackManifest.kt, ScriptPackScope.kt, ScriptPackTypes.kt, ServerPackCacheManager.kt
├── platform/     │ EntityRendererHooks.kt — platform-abstraction interfaces
├── registry/     │ KattonRegistry.kt (10 sub-registries), ReloadableBuiltInRegistry.kt, OwnershipTracker.kt, RegistryMutationUtil.kt
└── util/         │ Event.kt, Extension.kt, Result.kt, JResult.kt, ReflectUtil.kt, ScriptExecutionContext.kt, EntitySelectorBuilder.kt
```

### Paper Source Package Map

```
paper/src/main/
├── java/top/katton/paper/
│   ├── KattonPaperPlugin.java    │ Paper plugin entrypoint (JavaPlugin + Listener, 160 lines)
│   └── KattonPaperCommand.java   │ /katton command via Brigadier BasicCommand
├── kotlin/top/katton/
│   ├── paper/
│   │   ├── PaperNmsBridge.kt     │ Bukkit↔NMS type conversion (~30 methods, 275 lines)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Alumopper/Katton](https://github.com/Alumopper/Katton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
