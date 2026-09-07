---
trigger: always_on
description: This guide provides essential knowledge for AI coding agents working in the GrowERP codebase. It covers architecture, workflows, conventions, and integration points to maximize productivity and code quality.
---

# GrowERP AI Coding Agent Instructions

This guide provides essential knowledge for AI coding agents working in the GrowERP codebase. It covers architecture, workflows, conventions, and integration points to maximize productivity and code quality.

## 🏛️ Big Picture Architecture
GrowERP is an open-source, multi-platform ERP system designed to streamline business operations. It runs on Android, iOS, Web, Linux, and Windows, using Flutter for the frontend and Moqui for the backend. Comprehensive documentation and support are available at [https://www.growerp.com](https://www.growerp.com).

**Frontend Architecture** (Layered Building Blocks):
  1. **Building Blocks** (`growerp_*`): Domain-specific packages (`growerp_core`, `growerp_models`, `growerp_user_company`, `growerp_catalog`, etc.) that implement business logic and can be composed into applications
  2. **Applications**: Complete end-user apps (`admin`, `hotel`, `freelance`, `health`, `support`) built by composing building blocks
  3. **Package Dependencies**: Strict hierarchy - `growerp_models` (lowest) → `growerp_core` → domain packages → applications
  4. **State Management**: BLoC pattern (`flutter_bloc`) with centralized providers via `getCoreBlocProviders()` and domain-specific providers

**Backend Architecture** (Moqui Framework):
  - **Components**: Located in `moqui/runtime/component/` - `growerp` (core business logic), `mantle-udm` (universal data model), `mantle-usl` (universal service library)
  - **Multi-tenancy**: Built-in support for multi-company operations
  - **Data Flow**: REST APIs between Flutter frontend and Moqui backend services
  - **Integration**: Services defined in XML, entities in component definitions

## 🛠️ Developer Workflows

**Frontend (Melos Monorepo Management)**:
  ```bash
  dart pub global activate melos # one-time setup
  melos clean && melos bootstrap  # reset dependencies
  melos build                     # run build_runner on packages with build dependencies
  melos l10n                      # generate localizations for all packages
  melos watch                     # continuous build_runner watch
  ```

**Testing**:
  - **Local**: `melos test` (runs integration tests in dependency order)
  - **Headless CI**: `cd flutter && ./build_run_all_tests.sh` (Docker + Linux desktop + xvfb)
  - **Package Order**: Tests run in dependency order as defined in `melos.yaml`
  - **Test Structure**: Integration tests in `packages/*/example/integration_test/`
  - **Backend URL**: Injected via `--dart-define=BACKEND_URL=http://moqui` (no sed patching)

**Backend (Moqui Framework)** - All commands from `moqui/` directory:
  ```bash
  # Initial setup
  ./gradlew build
  java -jar moqui.war load types=seed,seed-initial,install no-run-es
  
  # Clean rebuild
  ./gradlew cleandb
  java -jar moqui.war load types=seed,seed-initial,install no-run-es
  
  # Start server
  java -jar moqui.war no-run-es  # port 8080 by default
  ```

## 📦 Project-Specific Conventions

**Package Architecture**:
  - **Strict Dependency Hierarchy**: `growerp_models` → `growerp_core` → domain packages (`growerp_catalog`, `growerp_user_company`, etc.) → applications (`admin`, `hotel`, etc.)
  - **Version Synchronization**: All `growerp_*` packages maintain synchronized versions (currently v1.9.0)
  - **Publishing Order**: Models/Core first, then domain packages, finally applications

**State Management (BLoC Pattern)**:
  - **Core Providers**: `getCoreBlocProviders()` in `growerp_core` sets up shared BLoCs (Auth, Chat, Theme, Locale, DataFetch, etc.)
  - **Domain Providers**: Each domain package provides additional BLoCs via functions like `getUserCompanyBlocProviders()`
  - **Message Pattern**: Use direct l10n keys with colon-delimited parameters (`'userAddSuccess:${user.name}'`) - see `QUICK_REFERENCE_BLOC_MESSAGES.md`

**Development Patterns**:
  - **Integration Testing**: Standardized setup in `example/integration_test/` with Docker emulator support
  - **Localization**: `flutter gen-l10n` generates type-safe l10n methods from `.arb` files
  - **Build Generation**: Uses `build_runner` for JSON serialization, Retrofit API clients, etc.
  - **Configuration**: `global_configuration` package loads from `app_settings.json`

## 🔗 Integration Points & External Dependencies

**API Communication**:
  - **REST Client**: Type-safe Retrofit client in `growerp_models` with Dio HTTP client
  - **WebSocket**: Real-time chat/notifications via `WsClient` (`web_socket_channel`)
  - **Authentication**: JWT tokens managed by `AuthBloc` across all requests
  - **Error Handling**: Centralized Dio error parsing with user-friendly messages

**Key Integrations**:
  - **Stripe**: Payment processing documented in `docs/Stripe_Payment_Processing_Documentation.md`
  - **Chat System**: WebSocket-based real-time messaging with `growerp_chat` package
  - **File Handling**: Image picker, file picker, PDF generation (`printing`, `pdf` packages)
  - **Localization**: Built-in support for multiple locales including Buddhist Era calendars

## 📚 Key Files & Directories

**Critical Configuration**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [growerp/growerp](https://github.com/growerp/growerp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
