---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Microbot Hub is a community plugin repository for the Microbot RuneLite client. It maintains a separation between core client functionality and community-contributed plugins, allowing rapid plugin development without affecting client stability. Each plugin is independently built, versioned, and packaged for GitHub Releases.

## Build System Architecture

The build system uses **Gradle with custom plugin discovery and packaging**:

- **Dynamic Plugin Discovery**: `build.gradle` scans `src/main/java/net/runelite/client/plugins/microbot/` for directories containing `*Plugin.java` files
- **Per-Plugin Source Sets**: Each discovered plugin gets its own Gradle source set, compile task, and shadow JAR task
- **Gradle Helper Scripts**: Core build logic lives in:
  - `gradle/project-config.gradle` - centralized configuration (JDK version, paths, GitHub release URLs, client version)
  - `gradle/plugin-utils.gradle` - plugin discovery, descriptor parsing, JAR creation, SHA256 hashing

### Build Commands

```bash
# Build all plugins
./gradlew clean build

# Build specific plugin(s) only (much faster for iteration)
./gradlew build -PpluginList=PestControlPlugin
./gradlew build -PpluginList=PestControlPlugin,AutoMiningPlugin

# Run tests (tests have access to all plugin source sets)
./gradlew test

# Generate plugins.json metadata file with SHA256 hashes (requires exact JDK 11)
./gradlew generatePluginsJson

# Copy plugin documentation to public/docs/
./gradlew copyPluginDocs

# Launch RuneLite debug session with plugins from Microbot.java
./gradlew run --args='--debug'

# Validate JDK version
./gradlew validateJdkVersion
```

## Plugin Structure

Each plugin lives in its own package under `src/main/java/net/runelite/client/plugins/microbot/<pluginname>/`:

```
<pluginname>/
├── <PluginName>Plugin.java    # Main plugin class with @PluginDescriptor
├── <PluginName>Script.java    # Script logic extending Script class
├── <PluginName>Config.java    # Configuration interface (optional)
├── <PluginName>Overlay.java   # UI overlay (optional)
└── Additional support classes
```

Matching resources under `src/main/resources/net/runelite/client/plugins/microbot/<pluginname>/`:

```
<pluginname>/
├── dependencies.txt           # Maven coordinates (optional)
└── docs/
    ├── README.md              # Plugin documentation
    └── assets/                # Screenshots, icons, etc.
```

## Plugin Descriptor Anatomy

Every plugin **must** have a `@PluginDescriptor` annotation with these **required** fields:

- `name` - Display name (use `PluginConstants.DEFAULT_PREFIX` or create custom prefix)
- `version` - Semantic version string (store in `static final String version` field)
- `minClientVersion` - Minimum Microbot client version required

Important **optional** fields:

- `authors` - Array of author names
- `description` - Brief description shown in plugin panel
- `tags` - Array of tags for categorization
- `iconUrl` - URL to icon image (shown in client hub)
- `cardUrl` - URL to card image (shown on website)
- `enabledByDefault` - Use `PluginConstants.DEFAULT_ENABLED` (currently `false`)
- `isExternal` - Use `PluginConstants.IS_EXTERNAL` (currently `true`)

Example:
```java
@PluginDescriptor(
    name = PluginConstants.MOCROSOFT + "Pest Control",
    description = "Supports all boats, portals, and shields.",
    tags = {"pest control", "minigames"},
    authors = { "Mocrosoft" },
    version = PestControlPlugin.version,
    minClientVersion = "1.9.6",
    iconUrl = "https://chsami.github.io/Microbot-Hub/PestControlPlugin/assets/icon.png",
    cardUrl = "https://chsami.github.io/Microbot-Hub/PestControlPlugin/assets/card.png",
    enabledByDefault = PluginConstants.DEFAULT_ENABLED,
    isExternal = PluginConstants.IS_EXTERNAL
)
@Slf4j
public class PestControlPlugin extends Plugin {
    static final String version = "2.2.7";
    // ...
}
```

## PluginConstants

The `PluginConstants.java` file is **shared across all plugins** (included in each JAR during build). It contains:

- Standardized plugin name prefixes (e.g., `DEFAULT_PREFIX`, `MOCROSOFT`, `BOLADO`)
- Global defaults: `DEFAULT_ENABLED = false`, `IS_EXTERNAL = true`

When creating a new plugin prefix, add it to `PluginConstants.java` for consistency.

## Adding External Dependencies

If a plugin needs additional libraries beyond the Microbot client:

1. Create `src/main/resources/net/runelite/client/plugins/microbot/<pluginname>/dependencies.txt`
2. Add Maven coordinates, one per line:
   ```
   com.google.guava:guava:33.2.0-jre
   org.apache.commons:commons-lang3:3.14.0
   ```
3. The build system automatically includes these in the plugin's shadow JAR

## Testing and Debugging Plugins


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chsami/Microbot-Hub](https://github.com/chsami/Microbot-Hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
