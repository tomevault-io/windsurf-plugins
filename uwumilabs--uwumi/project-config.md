---
trigger: always_on
description: - Expo + Expo Router app: screens live in `src/app/**` (route groups like `(tabs)` / `(settings)` and dynamic routes like `[mediaType].tsx`).
---

# Copilot instructions (Uwumi)

## Project map
- Expo + Expo Router app: screens live in `src/app/**` (route groups like `(tabs)` / `(settings)` and dynamic routes like `[mediaType].tsx`).
- Shared UI components: `src/components/**` (examples: `HorizontalTabs.tsx`, `CustomSelect.tsx`, `ui-primitives.tsx`).
- “Server state” lives in React Query hooks: `src/hooks/queries/**`.
- “App state” lives in Zustand stores (often MMKV-backed): `src/hooks/stores/**` and re-exported via `src/hooks/stores/index.ts` + `src/hooks/index.ts`.

## Data flow & providers (important)
- Providers are extension-driven for anime/movies: `useConsumetExtensions()` (see `src/hooks/stores/useExtensionStore.ts`) builds a `ProviderManager/ExtractorManager` to execute installed extension code.
- Provider selection + persistence is centralized in `src/constants/provider.ts` (`useProviderStore`, `DEFAULT_PROVIDERS`, `PROVIDERS`). `PROVIDERS` is derived from installed extensions at runtime.
- Manga providers are currently hard-mapped via `createProviderInstance()` in `src/constants/provider.ts`.
- Example query keys: `['anime','episodes',id,provider]`, `[mediaType,'info',id,metaProvider,type,provider]` (see `src/hooks/queries/infoQueries.ts`).


## Gesture + Sheet gotcha (Android)
- In `Sheet` UIs, keep interactive headers (e.g., `Tabs.List`) OUTSIDE any `Sheet.ScrollView` to avoid pan/scroll gestures stealing taps.
- Put scrolling on the body only: `Sheet.Frame` → header → `Sheet.ScrollView` for the content.

## Workflows (scripts)
- Dev: `npm run start` (uses `expo start --localhost`).
- Native: `npm run android` / `npm run ios`; for adb reverse + run use `npm run android:reverse`.
- Lint/format: `npm run lint`, `npm run format`.
- Tests: `npm run test` (Jest Expo).
- Theme generation: `npm run generate-themes`.
- Note: `postinstall` runs an Android Gradle task to download an AAR (`android/:app:downloadAar`).

## When editing
- Match existing patterns: memoized components (`memo`, `useCallback`, `useMemo`) are common in UI-heavy components.
- Prefer updating existing stores/hooks over inventing new state paths; keep query keys stable.# GitHub Copilot Instructions for Uwumi

## Project Overview
Uwumi is a React Native media streaming application built with Expo, supporting anime, manga, and movies. It features a modular extension system, custom theming engine, and offline capabilities.

## Architecture & Core Concepts

### 1. Extension System (`src/hooks/stores/useExtensionStore.ts`)
- **Dynamic Loading:** Extensions are downloaded from a remote registry and cached locally using `react-native-fs`.
- **Execution:** `react-native-consumet` manages the execution of these extensions to fetch media content.
- **Provider Management:** `src/constants/provider.ts` maps installed extensions to active providers.
- **Key Path:** `src/hooks/stores/useExtensionStore.ts` is the brain of this system.

### 2. Theming Engine (`src/themes/`)
- **CSS-Driven:** Themes are defined in CSS files (e.g., `src/themes/cloudflare.css`) using `@variant` blocks.
- **Generation:** A script (`src/scripts/generate-themes.ts`) parses these CSS files to generate type-safe TypeScript definitions in `src/themes/theme.ts`.
- **Usage:** Components consume themes via `useThemeStore` and `uniwind` styling.
- **Workflow:** **ALWAYS** run `npm run generate-themes` after modifying any `.css` file in `src/themes/`.

### 3. State Management
- **App State (Zustand):** Used for global UI state, settings, and extension management. Stores live in `src/hooks/stores/`.
- **Server State (React Query):** Used for all data fetching. Query keys follow the pattern `['mediaType', 'operation', ...params]`.
- **Persistence:** `react-native-mmkv` is used for high-performance synchronous storage.

### 4. Native Modules (`modules/`)
- The project uses custom Expo modules for platform-specific functionality:
  - `storage-permission-module`: Handles Android storage permissions.
  - `fullscreen-module`: Manages immersive mode.

## Critical Workflows

### Theme Updates
When adding or modifying a theme:
1. Edit the `.css` file in `src/themes/`.
2. Run `npm run generate-themes` to update `src/themes/theme.ts`.
3. Restart the dev server to see changes.

### Build & Run
- **Dev:** `npm run start` (uses `expo start --localhost`).
- **Android:** `npm run android` (builds native app).
- **Post-install:** The `postinstall` script downloads necessary AARs (e.g., ffmpeg-kit). Ensure this runs successfully.

## Coding Conventions

### UI & Styling
- **Library:** Uses `heroui-native` for core components (Card, Button, etc.) .
- **Styling:** Prefer `uniwind` classes (Tailwind-like).
- **Performance:** Heavily use `memo`, `useMemo`, and `useCallback` for UI components, especially in lists (`CustomFlashlist`).

### Data Fetching
- **Pattern:** Encapsulate queries in custom hooks (e.g., `useMediaInfo`, `useMediaFeed`).
- **Error Handling:** Use `sonner-native` for user-facing toast notifications on error.

### File Structure
- `src/app`: Expo Router screens.
- `src/modules`: Custom native modules.
- `src/themes`: CSS theme definitions.
- `src/scripts`: Build and utility scripts.

## Key Files to Know

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uwumilabs/uwumi](https://github.com/uwumilabs/uwumi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
