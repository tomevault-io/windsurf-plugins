---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Mostro Protocol Reference

The Mostro protocol documentation is **not** maintained in this repository. Always consult the official source:

- **Official docs**: https://mostro.network/protocol/
- **Repository**: https://github.com/MostroP2P/protocol

When implementing or debugging protocol-related features (order flows, actions, gift wrap messages, NIP integrations, etc.), refer to the official protocol documentation to verify how the protocol works. Do not rely on local copies or assumptions.

## Common Development Commands

### Build and Development
- `flutter pub get` - Install dependencies
- `flutter run` - Run the application 
- `dart run build_runner build -d` - Generate required files (localization, code generation)
- `flutter test` - Run unit tests
- `flutter test integration_test/` - Run integration tests
- `flutter analyze` - Static code analysis and linting
- `flutter format .` - Format code

### Code Generation
- Run `dart run build_runner build -d` after installing dependencies or updating localization files
- This generates files needed by `flutter_intl` and other code generators

### Essential Commands for Code Changes
- **`flutter analyze`** - ✅ **ALWAYS run after any code change** - Mandatory before commits
- **`flutter test`** - ✅ **ALWAYS run after any code change** - Mandatory before commits  
- **`dart run build_runner build -d`** - 🟡 **Only when code generation needed** (models, providers, mocks, localization)
- **`flutter test integration_test/`** - 🟡 **Only for significant changes** (core services, main flows)

## Architecture Overview

### State Management: Riverpod
- Uses **Riverpod** for dependency injection and state management
- Providers are organized by feature in `features/{feature}/providers/`
- **Notifier pattern** for complex state logic (authentication, order management)
- Notifiers encapsulate business logic and expose state via providers
- **SubscriptionManager Enhancement**: Includes manual initialization (`_initializeExistingSessions()`) to prevent orders getting stuck in previous states after app restart - protected by regression test

### Data Layer
- **Sembast** NoSQL database for local persistence
- Database initialization in `shared/providers/mostro_database_provider.dart`
- Repository pattern: All data access through repository classes in `data/repositories/`
- Models exported through `data/models.dart`

### Nostr Integration
- **NostrService** (`services/nostr_service.dart`) manages relay connections and messaging
- All Nostr protocol interactions go through this service
- **MostroFSM** (`core/mostro_fsm.dart`) manages order state transitions

### Navigation and UI
- **GoRouter** for navigation (configured in `core/app_routes.dart`)
- **flutter_intl** for internationalization (`l10n/` directory)
- Background services in `background/` for notifications and data sync

### Key Architecture Patterns
- Feature-based organization: `features/{feature}/{screens|providers|notifiers|widgets}/`
- Shared utilities and widgets in `shared/`
- Repository pattern for data access
- Provider pattern for dependency injection
- FSM pattern for order lifecycle management

### Relay Management System
- **Automatic Sync**: Real-time synchronization with Mostro instance relay lists via kind 10002 events
- **Manual Addition**: Users can add custom relays with strict validation (wss://, domains only, connectivity required)
- **Instance Validation**: Author pubkey checking prevents relay contamination between Mostro instances  
- **Two-tier Testing**: Nostr protocol + WebSocket connectivity validation
- **Memory Safety**: Isolated test instances protect main app connectivity during validation
- **Dual Storage Strategy**: Mostro/default relays stored in `settings.relays`, user relays stored in `settings.userRelays` with full metadata preservation
- **Source Tracking**: Relays tagged by source (user, mostro, default) for appropriate handling and storage strategy
- **Smart Re-enablement**: Manual relay addition automatically removes from blacklist, Mostro relay re-activation removes from blacklist during sync
- **URL Normalization**: All relay URLs normalized by removing trailing slashes to ensure consistent matching
- **Bootstrap Discovery Layer**: Hardcoded relays are NOT seeded into the visible list; `Config.bootstrapRelays` are defensive connections used only to discover a Mostro's kind 10002 (cold start via `NostrService.init` fail-safe, all-down via `RelayHealthMonitor` watchdog, instance switch via `ensureBootstrapConnectivity()`). Engaged additively (never `disconnectFromRelays()`); retirement is logical only since dart_nostr has no per-relay disconnect. See `docs/architecture/RELAY_SYNC_IMPLEMENTATION.md` ("Bootstrap Relay Discovery Layer")
- **Implementation**: Located in `features/relays/` with core logic in `RelaysNotifier`

#### Manual Relay Addition
- Users can manually add relays via `addRelayWithSmartValidation()` method
- Five sequential validations: URL normalization, duplicate check, domain validation, connectivity testing, blacklist management  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MostroP2P/mobile](https://github.com/MostroP2P/mobile) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
