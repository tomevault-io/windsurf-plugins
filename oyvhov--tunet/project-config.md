---
trigger: always_on
description: - React 18 + Vite Home Assistant dashboard. Real-time entity updates via `window.HAWS` WebSocket; dashboard config persists mostly to `localStorage`, while auth/session state also uses browser storage for same-browser reuse.
---

# Tunet Dashboard — Copilot Instructions

## Big picture

- React 18 + Vite Home Assistant dashboard. Real-time entity updates via `window.HAWS` WebSocket; dashboard config persists mostly to `localStorage`, while auth/session state also uses browser storage for same-browser reuse.
- **Architecture**:
  - **Data/Config**: Managed in `src/contexts` (`ConfigContext`, `PageContext`, `HomeAssistantContext`, `AppUiContext`, `ModalContext`).
  - **UI Orchestration**: `src/App.jsx` handles main layout, grid rendering, modal visibility state, and drag-and-drop.
  - **Modals**: Rendered via `ModalOrchestrator` in `src/rendering/`, controlled locally via `useModalState()` hook.

## Core data flow

1. **Init**: Read browser-stored Home Assistant config/auth state (`ha_url`, token-mode credentials, OAuth cache) via context/services.
2. **Connection**: `createConnection()` + `subscribeEntities()` updates global `entities` object.
3. **Usage**: Components consume config/entities via hooks. User changes persist immediately to browser storage and optional server-side profiles/settings APIs.

## Project structure

```
src/
  App.jsx                 # Main layout, grid rendering, modal managers
  main.jsx                # Entry point + ErrorBoundary
  styles/                 # CSS (index.css, dashboard.css, animations.css)
  config/                 # Pure data: constants, defaults, themes, onboarding
  icons/                  # Icon barrel (lucide re-exports + iconMap registry)
  utils/                  # Pure logic: formatting, cardUtils, gridLayout, dragAndDrop, logger
  contexts/               # Global state (ConfigContext, PageContext, HomeAssistantContext)
  hooks/                  # Custom React hooks
  services/               # HA WebSocket client, card actions, OAuth, Nordpool utils
  i18n/                   # Translation files (en.json, de.json, nb.json, nn.json, sv.json, zh.json)
  layouts/                # Header, StatusBar
  components/
    cards/                # Dashboard card widgets (SensorCard, LightCard, etc.)
    charts/               # Graphs & data viz (SparkLine, WeatherGraph, etc.)
    sidebars/             # Sidebar panels (Theme, Layout, Header)
    ui/                   # Shared primitives (M3Slider, IconPicker, ModernDropdown, etc.)
    pages/                # Full-page views (MediaPage, PageNavigation)
    effects/              # Visual effects (AuroraBackground, WeatherEffects)
  modals/                 # All dialogs (edit settings, device controls)
  rendering/              # Card renderer dispatch + ModalOrchestrator
  types/                  # TypeScript type definitions (dashboard.js)
  __tests__/              # Unit tests (vitest)

e2e/
  fixtures.js             # Playwright custom fixtures (mockHAConnection, authenticatedPage)
  oauth-flow.e2e.js       # OAuth authentication flow tests
  drag-and-drop.e2e.js    # Drag & drop interaction tests
  modals.e2e.js           # Modal interaction tests
  README.md               # E2E test documentation

playwright.config.js      # Playwright configuration (Chromium + Firefox)
E2E_TESTS_SETUP.md        # Comprehensive E2E setup guide
```

## Patterns & conventions

- **Card Data**: Generic cards (e.g., `GenericClimateCard`) read entity IDs from `cardSettings[settingsKey]`.
- **Sizing**: `settings.size` is `'small'|'large'`. Toggle capability checked via `canToggleSize()`.
- **Hooks**: `useEnergyData(entity, now)` expects a single entity object.
- **Icons**: selection stored as string names; mapped via `src/icons/iconMap.js`.
- **i18n**: keys in `src/i18n/{en,de,nb,nn,sv,zh}.json`. Setup is manual (no i18next).
- **Units (Metric/Imperial)**:
  - Never hard-code units in UI logic.
  - Read Home Assistant unit preferences from `useHomeAssistantMeta()` (`haConfig`) and resolve final mode with `getEffectiveUnitMode(unitsMode, haConfig)`.
  - Use shared helpers from `src/utils/units` (`inferUnitKind`, `convertValueByKind`, `getDisplayUnitForKind`, `formatUnitValue`) for display values.
  - For numeric sensor/modal values, convert from entity unit to active HA mode before rendering.
- **Styling**:
  - **Modals**: Use `.popup-surface` for boxed content (lists, groups) inside modals. Avoid manual `bg-[var(--glass-bg)]` where `.popup-surface` works.
  - **Cards**: Keep minimal. No heavy borders.
  - **Glassmorphism**: heavily used via CSS variables (`--glass-bg`, `--glass-border`).

## Performance optimizations (March 2026)

- **React.memo()**: All 21 card components wrapped with `React.memo()` to prevent unnecessary re-renders. This includes:
  - AlarmCard, CalendarCard, CameraCard, CarCard, CoverCard, FanCard
  - GenericAndroidTVCard, GenericClimateCard, GenericEnergyCostCard, GenericNordpoolCard
  - LightCard, MediaCards, MissingEntityCard, PersonStatus, RoomCard, SensorCard
  - SpacerCard, StatusPill, TodoCard, VacuumCard, WeatherTempCard
  - When adding new card components, ensure they are wrapped with `React.memo()` to maintain consistency
- **Code stats**: 51,937 total lines of code (src: 50,391, server: 1,014, scripts: 532)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oyvhov/Tunet](https://github.com/oyvhov/Tunet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
