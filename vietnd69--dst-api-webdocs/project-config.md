---
trigger: always_on
description: This rule defines the relationship between the [dst-scripts/](mdc:dst-api-webdocs/dst-scripts) source code directory and the [dst-api-webdocs/docs/game-scripts/](mdc:dst-api-webdocs/dst-api-webdocs/docs/game-scripts) documentation structure, establishing standards for cross-referencing and structural consistency.
---

# DST Scripts Documentation Structure

This rule defines the relationship between the [dst-scripts/](mdc:dst-api-webdocs/dst-scripts) source code directory and the [dst-api-webdocs/docs/game-scripts/](mdc:dst-api-webdocs/dst-api-webdocs/docs/game-scripts) documentation structure, establishing standards for cross-referencing and structural consistency.

## Directory Structure Mapping

The documentation structure mirrors the source code organization with enhanced categorization for better user navigation:

### Core Source Categories

| Source Directory | Documentation Directory | Purpose |
|------------------|------------------------|---------|
| [dst-scripts/components/](mdc:dst-api-webdocs/dst-scripts/components) | [docs/game-scripts/core-systems/](mdc:dst-api-webdocs/dst-api-webdocs/docs/game-scripts/core-systems) | Core game components organized by system type |
| [dst-scripts/behaviours/](mdc:dst-api-webdocs/dst-scripts/behaviours) | [docs/game-scripts/behaviours/](mdc:dst-api-webdocs/dst-api-webdocs/docs/game-scripts/behaviours) | AI behavior definitions |
| [dst-scripts/brains/](mdc:dst-api-webdocs/dst-scripts/brains) | [docs/game-scripts/brains/](mdc:dst-api-webdocs/dst-api-webdocs/docs/game-scripts/brains) | AI brain implementations |
| [dst-scripts/stategraphs/](mdc:dst-api-webdocs/dst-scripts/stategraphs) | [docs/game-scripts/stategraphs/](mdc:dst-api-webdocs/dst-api-webdocs/docs/game-scripts/stategraphs) | State machine definitions |
| [dst-scripts/prefabs/](mdc:dst-api-webdocs/dst-scripts/prefabs) | [docs/game-scripts/perfabs/](mdc:dst-api-webdocs/dst-api-webdocs/docs/game-scripts/perfabs) | Entity prefab definitions |
| [dst-scripts/widgets/](mdc:dst-api-webdocs/dst-scripts/widgets) | [docs/game-scripts/widgets/](mdc:dst-api-webdocs/dst-api-webdocs/docs/game-scripts/widgets) | UI widget implementations |
| [dst-scripts/screens/](mdc:dst-api-webdocs/dst-scripts/screens) | [docs/game-scripts/screens/](mdc:dst-api-webdocs/dst-api-webdocs/docs/game-scripts/screens) | Screen and UI definitions |
| [dst-scripts/map/](mdc:dst-api-webdocs/dst-scripts/map) | [docs/game-scripts/map/](mdc:dst-api-webdocs/dst-api-webdocs/docs/game-scripts/map) | World generation and map systems |
| [dst-scripts/util/](mdc:dst-api-webdocs/dst-scripts/util) | [docs/game-scripts/util/](mdc:dst-api-webdocs/dst-api-webdocs/docs/game-scripts/util) | Utility functions and helpers |
| [dst-scripts/tools/](mdc:dst-api-webdocs/dst-scripts/tools) | [docs/game-scripts/tools/](mdc:dst-api-webdocs/dst-api-webdocs/docs/game-scripts/tools) | Development and debugging tools |

### Special Documentation Organization

#### Core Systems Categorization
The [docs/game-scripts/core-systems/](mdc:dst-api-webdocs/dst-api-webdocs/docs/game-scripts/core-systems) directory organizes components from [dst-scripts/components/](mdc:dst-api-webdocs/dst-scripts/components) by functional categories:

```
core-systems/
├── character-systems/      # Character-related components
├── data-management/        # Data persistence and file operations
├── development-tools/      # Debugging and development utilities
├── fundamentals/          # Core framework components
├── game-configuration/    # Settings and configuration
├── game-mechanics/        # Gameplay feature components
├── localization-content/  # Translation and localization
├── mod-support/          # Modding framework components
├── networking-communication/ # Network and multiplayer
├── system-core/          # Engine integration components
├── user-interface/       # UI system components
└── world-systems/        # World generation and management
```

#### Root-Level Script Files
Scripts at [dst-scripts/](mdc:dst-api-webdocs/dst-scripts) root level are categorized by purpose:

| Script Category | Documentation Location | Examples |
|----------------|----------------------|----------|
| Character Speech | [docs/game-scripts/core-systems/character-systems/speech/](mdc:dst-api-webdocs/dst-api-webdocs/docs/game-scripts/core-systems/character-systems/speech) | `speech_*.lua` files |
| Configuration | [docs/game-scripts/core-systems/game-configuration/](mdc:dst-api-webdocs/dst-api-webdocs/docs/game-scripts/core-systems/game-configuration) | `tuning.lua`, `worldsettings_overrides.lua` |
| Core Utilities | [docs/game-scripts/core-systems/fundamentals/utilities/](mdc:dst-api-webdocs/dst-api-webdocs/docs/game-scripts/core-systems/fundamentals/utilities) | `util.lua`, `simutil.lua` |
| Debug Tools | [docs/game-scripts/core-systems/development-tools/debugging/](mdc:dst-api-webdocs/dst-api-webdocs/docs/game-scripts/core-systems/development-tools/debugging) | `debugcommands.lua` |

## Source Code Reference Standards

### Function and Code References

When documenting functions or code patterns, always reference the actual source file:

```markdown
### FunctionName(param1, param2) {#function-name}

**Source:** [dst-scripts/components/health.lua](mdc:dst-api-webdocs/dst-scripts/components/health.lua)

**Description:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vietnd69/dst-api-webdocs](https://github.com/vietnd69/dst-api-webdocs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
