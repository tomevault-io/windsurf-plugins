---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## TL;DR (What this repo is)
Spigradle is a Gradle plugin for developing Spigot, Bungeecord, and NukkitX plugins.
It provides:
- Automatic plugin.yml / bungee.yml generation
- Main class detection (ASM bytecode scan)
- Dependency and repository shortcuts (via catalogs and DSL)
- Debug tasks (Spigot only)

---

## Non-negotiables (DO / DON'T)

### DO
- Use the project’s Gradle workflows below (build/test/docs).
- Treat `debugSpigot { }` as an **extension**, not a task (task is `debug${ProjectName}`).
- For Markdown docs: edit templates, then run `updateTemplateDocs`.
- Add the required copyright header to **all new source files**.

### DON'T
- Do not edit generated Markdown files directly (`docs/*.md`, root generated docs).
- Do not assume `SpigotExtension.load` is an enum: it’s `Property<String>`.

---

## Quick commands (copy/paste)

### Build & test
```bash
# Build
./gradlew assemble

# Tests (runs with 8 parallel forks)
./gradlew test

# Publish to local Maven repository
./gradlew publishToMavenLocal
```

**Important**: Tests depend on `publishToMavenLocal`, so the plugin is automatically published locally before running tests.

### Documentation
```bash
# Update documentation from templates
./gradlew updateTemplateDocs
```
This processes:
- `docs/templates/` → `docs/*.md`
- `docs/root-templates/` → `*.md` (project root)

### Version management
Project version is defined in `gradle.properties`.

---

## Repo map (modules)

Multi-module Gradle project with composite builds:

```
spigradle/
├── plugin/                    # Main Gradle plugin module
│   └── src/main/kotlin/io/typst/spigradle/
├── spigot-catalog/            # Version Catalog for Spigot dependencies
├── bungee-catalog/            # Version Catalog for BungeeCord dependencies
├── nukkit-catalog/            # Version Catalog for NukkitX dependencies
├── common-catalog/            # Version Catalog for common dependencies
├── spigot-bom-1.16/           # BOM for Spigot 1.16 deps (placeholder)
├── spigot-bom-1.20/           # BOM for Spigot 1.20 deps (placeholder)
└── build-logic/               # Composite build: convention plugins
    ├── catalog/               # Version Catalog generation plugin
    ├── central-publish/       # Maven Central publication
    ├── docs/                  # Dokka documentation configuration
    ├── java/                  # Java toolchain configuration
    └── publish/               # Gradle Plugin Portal publication
```

### Module descriptions
- `plugin`: Main Gradle plugin (Spigot/Bungee/Nukkit). Published as `io.typst:spigradle`.
- `spigot-catalog`: Version Catalog for Spigot-related deps + Spigradle coords. Published as `io.typst:spigot-catalog`.
  - Configured via `SpigradleCatalogPublishPlugin` in `build-logic/catalog/`.
  - Includes all deps from `PaperDependencies` enum and Spigradle plugin coordinates.
- `bungee-catalog`: Published as `io.typst:bungee-catalog`.
- `nukkit-catalog`: Published as `io.typst:nukkit-catalog`.
- `common-catalog`: Common deps (Lombok, jOOQ, HikariCP, etc). Published as `io.typst:common-catalog`.
  - Includes deps from `CommonDependencies` enum.
- `spigot-bom-*`: Placeholder modules (not implemented yet).
- `build-logic`: Internal convention plugins for building this repo. Catalog versions are independently versioned in `gradle.properties`.

---

## Plugin IDs (public surface)

### Main plugins
1. `io.typst.spigradle.spigot` → `io.typst.spigradle.spigot.SpigotPlugin`
2. `io.typst.spigradle.bungee` → `io.typst.spigradle.bungee.BungeePlugin`
3. `io.typst.spigradle.nukkit` → `io.typst.spigradle.nukkit.NukkitPlugin`

### Base plugins (extensions/repository DSL only; no YAML generation)
4. `io.typst.spigradle.spigot-base` → `io.typst.spigradle.spigot.SpigotBasePlugin`
5. `io.typst.spigradle.bungee-base` → `io.typst.spigradle.bungee.BungeeBasePlugin`
6. `io.typst.spigradle.nukkit-base` → `io.typst.spigradle.nukkit.NukkitBasePlugin`

### Planned (not yet released)
- `io.typst.spigradle.paper` → `io.typst.spigradle.paper.PaperPlugin`
- `io.typst.spigradle.paper-base` → `io.typst.spigradle.paper.PaperBasePlugin`

---

## Core architecture (where to look)

### Main class detection (ASM)
- Task: `SubclassDetection`
  - `plugin/src/main/kotlin/io/typst/spigradle/SubclassDetection.kt`
- Uses ASM 9.9 with flags: `SKIP_CODE`, `SKIP_DEBUG`, `SKIP_FRAMES`
- Detection targets:
  - Spigot: `org/bukkit/plugin/java/JavaPlugin`
  - Bungee: `net/md_5/bungee/api/plugin/Plugin`
  - Nukkit: `cn/nukkit/plugin/PluginBase`

Detection algorithm:
1. Scan all `.class` files; read name/superclass/interfaces/modifiers
2. Build a directed inheritance graph in `DetectionContext`
3. Traverse to find classes inheriting from the platform base class
4. Filter to non-abstract, public class as main class
5. Write FQCN (dot notation) to output file

Detection framework:
- `plugin/src/main/kotlin/io/typst/spigradle/detection/`
  - `ClassDefinition.kt` - Class metadata
  - `DetectionContext.kt` - Detection state + graph
  - `DirectedGraph.kt` - Graph utilities

### YAML generation
- Task: `YamlGenerate`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [typst-io/spigradle](https://github.com/typst-io/spigradle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
