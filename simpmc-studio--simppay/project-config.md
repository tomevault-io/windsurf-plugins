---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SimpPay is a Minecraft plugin for automated Vietnamese payment processing (QR banking via PayOS/Web2M/Sepay and prepaid card recharging via TheSieuToc, Gachthe1s, Card2K, TheSieuRe, Doithe1s). Built for Paper 1.13-1.21.4+ servers using Java 8.

**Gradle structure:**
- `simppay-paper`: Main plugin implementation (contains all core code)
- `simppay-converter`: Data conversion utilities (minimal)

## Build Commands

```bash
./gradlew build                    # Build all, output: build/libs/SimpPay-<version>.jar
./gradlew clean build              # Clean build
./gradlew :simppay-paper:shadowJar # Create shaded JAR only
```

**Output location:** `build/libs/SimpPay-<version>.jar` (can override with `OUTPUT_DIR` env var)

**Dependencies:** All shaded dependencies are relocated under `me.typical.lib.*`:
- ConfigLib → `me.typical.lib.configlib`
- CommandAPI → `me.typical.lib.commandapi`
- FoliaLib → `me.typical.lib.folialib`

**Paper Plugin Loader:** Uses `SimpPayLoader` (implements `PluginLoader`) to download dependencies at runtime via Maven repositories. This ensures compatibility with Paper 1.21+ plugin loading system. Dependencies are fetched from:
- Maven Central (Google mirror)
- XenonDevs (InvUI)
- SpaceIO (AnvilGUI snapshots)

**Runtime-loaded libraries:**
- InvUI 1.49
- OkHttp 5.0.0-alpha.12
- ConfigLib 4.8.0
- ORMLite 6.1
- HikariCP 6.3.0
- H2 Database 2.3.232
- Lombok 1.18.34
- Commons Codec 1.18.0
- AnvilGUI 2.0.3-SNAPSHOT

## Core Architecture

### Service Pattern
All major functionality implements `IService` interface with `setup()` and `shutdown()` methods. Services are registered in `SPPlugin.onEnable()` and auto-register as event listeners if they implement `Listener`.

**Registration order (important for dependency resolution):**
1. `OrderIDService` - Sequential payment ID generation
2. `BankCacheService` - Async VietQR bank data cache (fetched on startup)
3. `CacheDataService` - In-memory leaderboard cache with 1-minute TTL
4. `DatabaseService` - ORMLite DAOs and sub-services
5. `PaymentService` - Active payment tracking and handler routing
6. `MilestoneService` - Milestone tracking with BossBar display
7. `WebhookService` - HTTP server for Sepay webhook callbacks (port 8080 by default)

Access: `SPPlugin.getService(ServiceClass.class)`

### Handler System
Payment gateways use the `PaymentHandler` interface. Handlers are dynamically instantiated via reflection from enum types in `HandlerRegistry`:
- `CardAPI` enum → Card handlers (`TSTHandler`, `GT1SHandler`, `Card2KHandler`, `TSRHandler`, `DT1SHandler`)
- `BankAPI` enum → Bank handlers (`PayosHandler`, `W2MHandler`, `SepayHandler`)
- `CoinsAPI` enum → Points handlers (`PlayerPointsHandler`, `CoinsEngineHandler`, `DefaultCoinsHandler`)

Each enum specifies its handler class reference. Handlers are loaded during `setup()` and on config reload.

### Configuration System
ConfigLib with YAML files managed by `ConfigManager`:
- Config classes in `config/types/` with `@Folder("subfolder")` for subdirectories
- Kebab-case YAML keys from camelCase field names (`NameFormatters.LOWER_KEBAB_CASE`)
- Custom serializers for Adventure API types (`Key`, `Sound`)
- Access: `ConfigManager.getInstance().getConfig(ConfigClass.class)`
- Register new configs in `ConfigManager.configClasses` list

**Key configs:**
- `MainConfig` - Core plugin settings (debug mode, locale)
- `MessageConfig` - MiniMessage-formatted player messages
- `DatabaseConfig` - H2/MySQL/MariaDB connection settings
- `StreakConfig` - Consecutive day rewards configuration
- `MilestonesPlayerConfig` / `MilestonesServerConfig` - Milestone rewards with `MilestoneEntry`
- `NaplandauConfig` - First-time recharge rewards
- Banking configs: `PayosConfig`, `Web2mConfig`, `SepayConfig`
- Card configs: `ThesieutocConfig`, `Card2kConfig`, `Gachthe1sConfig`, etc.

### UI System
Uses **InvUI** (v1.49) for chest GUIs (static `openMenu()` methods) and AnvilGUI for text input:

**Menu flow:**
```
CardListView.openMenu(player)
  → Shows enabled card types
  → Clicks open CardPriceView.openMenu(player, cardType)
    → Shows price options (10k, 20k, 50k, etc.)
    → Clicks open CardSerialInput (AnvilGUI)
      → User enters serial number
      → Opens CardPinInput (AnvilGUI)
        → User enters PIN
        → Initiates payment via PaymentService
```

**Other menus:**
- `PaymentHistoryView.openMenu(player, playerName)` - Async-loaded paginated transaction history with `PagedGui`
- `StreakMenuView.openMenu(player)` - Streak progress display with milestone rewards

Menu layouts configured via `DisplayItem` in `menus/*.yml` configs.

### Commands
CommandAPI (v11.1.0) managed by `CommandHandler`:
- `/napthe` - Open card recharge GUI (permission: `simppay.napthe`)
- `/napthenhanh <card> <price> <serial> <pin>` - Quick recharge without GUI (permission: `simppay.napthenhanh`)
- `/bank <amount>` - QR banking recharge (permission: `simppay.banking`)
- `/lichsunapthe [player]` - Payment history (permission: `simppay.lichsunapthe`)
- `/streak` - Streak menu (permission: `simppay.streak`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SimpMC-Studio/SimpPay](https://github.com/SimpMC-Studio/SimpPay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
