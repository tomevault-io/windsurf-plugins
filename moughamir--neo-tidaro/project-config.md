---
trigger: always_on
description: This document provides essential context for AI models and contributors working in this project. Adhering to these guidelines ensures consistency, modularity, and long-term maintainability.
---


# GEMINI.md: AI Collaboration Guide

This document provides essential context for AI models and contributors working in this project. Adhering to these guidelines ensures consistency, modularity, and long-term maintainability.

## 1. Project Overview & Purpose

- **Primary Goal:**
  This repository is a **multi-package Flutter/Dart monorepo** managed with **Melos**, whose configuration is integrated directly into **pubspec.yaml**. It enables modular development where apps, features, and core libraries coexist in a unified workspace.

The primary target is **Tidaro** — a housekeeping and service mediation platform designed for Morocco and scalable to global markets. The app is **mobile-first**, with emphasis on **multi-language accessibility** (Arabic, Tifinagh, Berber Latin, English, French).

### Real-World Goal

Tidaro connects households and businesses with vetted housekeeping professionals in Morocco.

- Customers can discover services, get transparent pricing, book and manage appointments, pay securely, and rate experiences.
- Providers (professionals/agencies) can manage availability, assignments, routes, and performance, and receive payouts.
- Operators use the dashboard (TiDash) to oversee supply/demand, staff, bookings, service quality, and support.

Scope across apps:

- `apps/tidaro/`: customer-facing booking experience.
- `apps/tidash/`: operational dashboard for staff, bookings, metrics.
- `apps/tidaro_mini/`: lightweight companion app variant for rapid testing or limited deployments.

Success indicators:

- Reduced booking friction (time-to-book, cancellation rate).
- High fulfillment rate and on-time completion.
- Provider retention and utilization.
- CSAT/NPS and low support load via reliable flows.
- Secure, compliant data handling using Supabase (Auth, Storage, Realtime, Functions).

See [[#Business KPIs]] for formal tracking metrics.

## 2. Core Technologies & Stack

- **Languages:** Dart (latest stable supported by Flutter).
- **Frameworks & Platforms:** Flutter (Android, iOS, Web, Linux, macOS, Windows).
- **Backend & Database:** Supabase/PostgreSQL (Auth, Storage, Functions, Realtime).
- **State Management:** Redux (standard across workspace). Existing Redux integrations are the reference; new features should use Redux.
- **Internationalization:** `packages/languist` with `languist.yaml` and Flutter gen-l10n.
- **Containerization:** Docker Compose for local services where applicable.
- **Monorepo Tooling:**

  - [`melos`](https://melos.invertase.dev) → workspace manager (configured in `pubspec.yaml`)
  - `dart pub` → package dependencies
  - `pnpm` → JavaScript package manager for root-level scripts (e.g., seeding)

## 3. Workspace Structure

This project follows a **Melos workspace** layout, with configuration embedded in `pubspec.yaml`.

```shell
.
├── apps/                     # App entrypoints
│   ├── tidaro/               # Main app
│   ├── tidaro_mini/          # Lightweight app variant
│   └── tidash/               # Admin/dashboard app
│
├── packages/                 # Reusable and shared packages
│   ├── core/                 # Core utilities and base functionality
│   ├── shared/               # State mgmt, utilities, shared functions
│   ├── languist/             # Localization (ARB, gen:l10n, config)
│   └── device_sensors/       # Sensors and device integrations
│
├── examples/                 # Minimal runnable examples
│   ├── auth_flow/
│   ├── basic_app/
│   └── ui_kit_showcase/
│
├── supabase/                 # DB schema, seeds, config
│   ├── migrations/
│   └── seeds/
│
├── documentation/            # Architecture, workflow, guides
│   ├── 01-workspace-structure.md
│   ├── 02-coding-conventions.md
│   ├── 03-development-workflow.md
│   ├── 04-packages-overview.md
│   └── 05-platform-support.md
│
├── pubspec.yaml              # Workspace-level dependencies, scripts, and Melos configuration
├── pubspec.yaml              # Workspace-level dependencies and scripts
└── .github/workflows/        # CI pipelines
```

**Philosophy:**

- `/apps/` → Thin entrypoints that compose features and present UI.
- `/packages/` → Independent, reusable modules (domain, data, shared UI).
- Business logic lives in packages; apps wire features and screens.

## 4. Coding Conventions & Style Guide

- **Formatting:**

  - Use `dart format` (2-space indentation, trailing commas).
  - Must pass `flutter analyze`.

- **Naming Conventions:**

  - Variables, functions: `camelCase`
  - Classes, Widgets: `PascalCase`
  - Files: `snake_case.dart`

- **Architecture & State Management:**

  - Clean Architecture and separation of concerns.
  - State management: Use Redux for new and existing features. Keep reducers, actions, selectors, and middleware in `packages/shared/` (or domain-specific packages) and compose them at app level.
  - Networking/services/models live in `packages/core/` or feature packages; UI consumes via abstractions.
  - Localization via `packages/languist` and generated l10n delegates.

- **Error Handling:**

  - Always handle async errors (`try/catch`, `FutureBuilder`, `AsyncSnapshot`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/moughamir) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
