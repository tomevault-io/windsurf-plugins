---
trigger: always_on
description: **SPART Board** is an interactive, widget-based classroom management application built with **React 19**, **TypeScript**, and **Vite**. It leverages **Firebase** for backend services (Authentication, Firestore, Storage) and features a drag-and-drop interface for various classroom tools (timers, polls, noise meters, etc.). The development environment includes access to the **Firebase CLI** for rule and index deployment.
---

# Gemini Context: SPART Board

## Project Overview

**SPART Board** is an interactive, widget-based classroom management application built with **React 19**, **TypeScript**, and **Vite**. It leverages **Firebase** for backend services (Authentication, Firestore, Storage) and features a drag-and-drop interface for various classroom tools (timers, polls, noise meters, etc.). The development environment includes access to the **Firebase CLI** for rule and index deployment.

**Key Technologies:**

- **Frontend:** React 19, TypeScript, Vite, Tailwind CSS (Custom Brand Theme), Lucide React
- **Backend:** Firebase (Auth, Firestore, Storage)
- **AI:** Gemini API (via `@google/genai`)
- **Package Manager:** pnpm
- **Testing:** Vitest (Unit), Playwright (E2E), Istanbul (Coverage)

## Project Structure & Architecture

**CRITICAL NOTE:** This project uses a **flat-ish file structure**. Most source code resides at the project root (e.g., `App.tsx`, `types.ts`, `index.tsx`), but specialized logic is organized into directories like `/components/`, `/context/`, `/hooks/`, `/utils/`, and `/config/`. There is **NO `src/` directory** for primary application code (any existing `src/` is for temporary scratch files or isolated testing).

### Directory Layout

- `@/` (Root Alias): Maps to the project root directory.
- `/components/`: React components.
  - `admin/`: Admin-specific components (`AdminSettings.tsx`, `FeaturePermissionsManager.tsx`).
  - `auth/`: Authentication UI (`LoginScreen.tsx`).
  - `common/`: Reusable components (e.g., `DraggableWindow.tsx`, `Button.tsx`).
  - `layout/`: Core layout (`Sidebar.tsx`, `Dock.tsx`, `DashboardView.tsx`).
  - `widgets/`: Individual widget implementations and `WidgetRenderer.tsx`.
    - `random/`: Specialized sub-folder for the refactored Random widget.
    - `MaterialsWidget/`: Specialized sub-folder for the Materials widget.
- `/context/`: React Context providers (`AuthContext`, `DashboardContext`).
- `/hooks/`: Custom hooks (`useFirestore`, `useStorage`).
- `/config/`: Configuration files (`firebase.ts`, `tools.ts`, `widgetGradeLevels.ts`).
- `/utils/`: Shared utility functions.
- `/functions/`: Firebase Cloud Functions (backend logic).
- `/tests/`: Test setup and E2E tests.
  - `e2e/`: Playwright end-to-end tests.
- `/types.ts`: Global TypeScript definitions.
- `/App.tsx`: Root application component.
- `/index.tsx`: Entry point.
- `/Dockerfile` & `/nginx.conf`: Infrastructure for containerized deployment.

### Architecture Patterns

- **Lazy Loading:** To optimize initial bundle size, all widgets and their settings panels are lazily loaded via `React.lazy` and managed in `components/widgets/WidgetRegistry.ts`.
- **State Management:** Centralized via React Context (`DashboardContext`, `AuthContext`).
- **Widget System:** Plugin-based. New widgets are added to `components/widgets/`, registered in `types.ts` (`WidgetType`) and `config/tools.ts` (`TOOLS`), and mapped in `WidgetRenderer.tsx`.
- **Grade Level Filtering:** Widgets are categorized by grade levels (K-2, 3-5, 6-8, 9-12) in `config/widgetGradeLevels.ts`.
- **Feature Permissions:** Access to widgets can be toggled and restricted to admins or beta users via the `FeaturePermission` system and managed in `AdminSettings`.
- **Weather Logic:** Admins can configure custom temperature ranges (Above, Below, or Range) in `FeaturePermissionsManager.tsx`, which the `WeatherWidget` uses to display conditional messages and images.
- **CORS Proxy Logic:** Certain widgets (e.g., `WeatherWidget`, `LunchCountWidget`) use a multi-proxy fallback mechanism to bypass cross-origin restrictions for third-party APIs.
- **Data Persistence:**
  - **Primary:** Firestore (real-time sync).
  - **Fallback:** `localStorage` (migrated to Firestore on sign-in).
- **Security:** Firestore Security Rules enforce ownership (users can only edit their own dashboards) and admin privileges.

## Design System & Styling

- **Typography**: **Lexend** (UI), **Patrick Hand** (Accents), **Roboto Mono** (Code).
- **Brand Colors**: Custom brand colors configured in `tailwind.config.js`:
  - **Brand Blue**: Primary (#2d3f89), Dark (#1d2a5d), Light (#4356a0), Lighter (#eaecf5).
  - **Brand Red**: Primary (#ad2122), Dark (#7a1718), Light (#c13435), Lighter (#e5c7c7).
  - **Brand Gray**: A semantic grayscale palette (darkest to lightest) for UI consistency.
- **Theme Config**: See `tailwind.config.js` for full palette and extensions.

## Slash Commands

The following slash commands are available to streamline the development workflow:

- **/clean**: Deletes all unsaved work to return to the last saved state.
  - _Action:_ `git reset --hard HEAD` and `git clean -fd`.
  - _Caution:_ Permanently deletes changes since the last `/preview`.
- **/preview**: Saves your current work and updates your online preview.
  - _Action:_ Stages, commits, and pushes changes to the current branch.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OPS-PIvers) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
