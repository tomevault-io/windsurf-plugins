---
trigger: always_on
description: Welcome to the `equran-app` codebase. This document serves as a dense, clear, structural onboarding guide and system map for developers and AI agents. It maps the directories, architectural paradigms, subsystem designs, and coding guardrails that must be strictly followed.
---

# Codebase Architecture Map (AGENTS.md)

Welcome to the `equran-app` codebase. This document serves as a dense, clear, structural onboarding guide and system map for developers and AI agents. It maps the directories, architectural paradigms, subsystem designs, and coding guardrails that must be strictly followed.

---

## 🏗️ Architectural Paradigm

### Feature-First Modular Structure
This codebase does not follow a strict traditional horizontal layered layout (e.g. `/lib/src/models`, `/lib/src/repositories`, etc.). Instead, it uses a **Feature-First Modular Architecture**:
- Each self-contained feature is co-located in its own top-level directory directly under `/lib/` (e.g., `lib/prayer/`, `lib/duas/`, `lib/hifz/`).
- Within a feature directory, you will find co-located files including pages/views, repositories, services, local models, and specialized widgets (e.g., `lib/hifz/pages/` and `lib/hifz/models/`).
- Global and shared layers live in centralized utility directories:
  - `lib/backend/`: Core databases, app settings, and network/caching logic.
  - `lib/theme/`: Global typography and multi-theme styling configurations.
  - `lib/widgets/`: Shared reusable UI elements.
  - `lib/utils/`: Raw calculations, parsing, and formatting logic.

### State Management & Reactive UI
- **ValueNotifier Pattern:** This project does not use state management libraries like `flutter_bloc` or `riverpod`. Instead, it is built on native Flutter `ValueNotifier`, `ChangeNotifier`, and `ValueListenableBuilder` widgets.
- **Custom Reactive Streams ("Blocs"):** Core global state is managed via lightweight classes that extend or use `ValueNotifier` to stream state updates (e.g., `NavigationBloc` inside [navigation_bloc.dart](file:///home/yousuf/Documents/Personal%20Projects/equran-app/lib/backend/navigation_bloc.dart) extends `ValueNotifier<NavigationState>`).
- **UI Consumption:** Views subscribe to updates reactively using `ValueListenableBuilder` or custom state streams to rebuild only the atomic components that depend on changed variables, keeping rendering overhead minimal.

---

## 📁 Codebase Directory Map

### Shared Core & Shared Utilities
- **[lib/backend/](file:///home/yousuf/Documents/Personal%20Projects/equran-app/lib/backend/)** -> Core databases, repository layers, and background services:
  - Database helpers (Hive/SQLite-based) such as `bookmark_db.dart`, `hifz_db.dart`, `settings_db.dart`, `surah_db.dart`.
  - Global app settings and backup services (`settings_db.dart`, `backup_service.dart`).
  - Font loading engines (`qpc_v4_font_service.dart`) and font patch utilities (`qcf_cpal_patcher.dart`).
  - Downloader services (`resource_download_service.dart`, `resource_install_store.dart`, `audio_downloads.dart`).
- **[lib/theme/](file:///home/yousuf/Documents/Personal%20Projects/equran-app/lib/theme/)** -> Global design tokens:
  - Colors, shapes, and theme mode listeners.
  - Typography map [equran_text_styles.dart](file:///home/yousuf/Documents/Personal%20Projects/equran-app/lib/theme/equran_text_styles.dart) which resolves font families dynamically.
- **[lib/widgets/](file:///home/yousuf/Documents/Personal%20Projects/equran-app/lib/widgets/)** -> Reusable atomic UI components (e.g., progress indicators, selection dialogs, and main reading player sheets).
- **[lib/utils/](file:///home/yousuf/Documents/Personal%20Projects/equran-app/lib/utils/)** -> Formatting tools, recitation profile indices (`reciter.dart`), and shared validation logic.
- **[lib/services/](file:///home/yousuf/Documents/Personal%20Projects/equran-app/lib/services/)** -> Platform-level configurations and system handlers (e.g., frame rate policies).

### Feature Modules (Co-located Code)
- **[lib/prayer/](file:///home/yousuf/Documents/Personal%20Projects/equran-app/lib/prayer/)** -> Prayer times and Qibla module. Contains pages, timezone/notifications/location service logic, and regional calculators.
- **[lib/duas/](file:///home/yousuf/Documents/Personal%20Projects/equran-app/lib/duas/)** -> Supplications (Hisn al-Muslim), Asma ul-Husna page, Tasbih digital counter, and favorite bookmark lists.
- **[lib/hifz/](file:///home/yousuf/Documents/Personal%20Projects/equran-app/lib/hifz/)** -> Quran memorization tracker, containing scheduling models, logs, progress tracking algorithms, and session views.
- **[lib/reading_plans/](file:///home/yousuf/Documents/Personal%20Projects/equran-app/lib/reading_plans/)** -> Modular Quran reading plan generation, schedule logs, and tracking algorithms.
- **[lib/search/](file:///home/yousuf/Documents/Personal%20Projects/equran-app/lib/search/)** -> Quran text index searcher, pagination handlers, and search result views.
- **[lib/zakat/](file:///home/yousuf/Documents/Personal%20Projects/equran-app/lib/zakat/)** -> Interactive Zakat calculation utility.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ya27hw/equran_app](https://github.com/ya27hw/equran_app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
