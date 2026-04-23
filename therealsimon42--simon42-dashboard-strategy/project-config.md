---
trigger: always_on
description: Custom Lovelace Dashboard Strategy for Home Assistant. Generates dynamic dashboards from area/device/entity metadata with flexible user configuration. This project is actively used by Simons loved YouTube viewers — clean, stable code is top priority.
---

# Simon42 Dashboard Strategy

Custom Lovelace Dashboard Strategy for Home Assistant. Generates dynamic dashboards from area/device/entity metadata with flexible user configuration. This project is actively used by Simons loved YouTube viewers — clean, stable code is top priority.

## Architecture

**Language:** TypeScript (ES2020, strict mode)
**Build:** Webpack → code-split chunks (main + lit + core + views + editor on-demand)
**Distribution:** HACS-compatible (Custom Repository), deployed to `/config/www/community/simon42-dashboard-strategy/`

### Module Overview

```
src/
├── simon42-dashboard-strategy.ts    # Entry point: generate(config, hass) → {title, views[]}
├── Registry.ts                      # Singleton registry (synchronous init from hass object, pre-computed Maps)
├── types/                           # Type definitions
│   ├── homeassistant.ts             #   HA interfaces (hass object, callWS, formatters)
│   ├── registries.ts                #   Entity/device/area/floor registry types
│   ├── strategy.ts                  #   Simon42 config types
│   └── lovelace.ts                  #   Lovelace card/view/section/badge types
├── utils/
│   ├── entity-filter.ts             #   Entity collection (collectPersons, findWeatherEntity, findDummySensor)
│   ├── name-utils.ts                #   Name/entity helpers (stripAreaName, getVisibleAreas, sortByLastChanged)
│   ├── badge-builder.ts             #   Person badge creation
│   └── view-builder.ts              #   View generation (overview, utility, area views)
├── sections/
│   ├── OverviewSection.ts           #   Clock, alarm, search, summaries, favorites
│   ├── AreasSection.ts              #   Area cards (with optional floor grouping)
│   └── WeatherEnergySection.ts      #   Weather forecast + energy distribution
├── cards/                           # LitElement custom cards (reactive, tile card pooling)
│   ├── SummaryCard.ts               #   Reactive summary tiles (lights, covers, security, batteries, climate)
│   ├── LightsGroupCard.ts           #   On/off light grouping (heading badges + tile card pool + floor grouping)
│   └── CoversGroupCard.ts           #   Open/closed cover grouping (heading badges + tile card pool)
├── views/                           # Specialized view strategies
│   ├── RoomViewStrategy.ts          #   Room detail view (15+ device classes, Reolink + Aqara cameras)
│   ├── LightsViewStrategy.ts        #   Light aggregation (optional floor grouping)
│   ├── CoversViewStrategy.ts        #   Cover/blind aggregation
│   ├── SecurityViewStrategy.ts      #   Security overview (locks, doors, windows, garages)
│   ├── BatteriesViewStrategy.ts     #   Battery status (critical/low/ok)
│   └── ClimateViewStrategy.ts       #   Climate/thermostat overview (heating/cooling/idle/off)
└── editor/                          # Configuration UI
    ├── StrategyEditor.ts            #   Editor class (largest file — config form, state management)
    ├── editor-handlers.ts           #   Event listeners, drag/drop area reordering
    ├── editor-template.ts           #   HTML template generation
    └── editor-styles.ts             #   CSS styling
```

Output:
```
dist/
├── simon42-dashboard-strategy.js                        # Entry point (instant custom element registration)
├── simon42-dashboard-strategy-core.<hash>.js            # Registry, cards, utils
├── simon42-dashboard-strategy-lit.<hash>.js             # Lit framework (shared)
├── simon42-dashboard-strategy-views.<hash>.js           # All view strategies
├── simon42-dashboard-strategy-editor.<hash>.js          # Editor (lazy-loaded on demand)
├── *.js.gz / *.js.br                                    # Pre-compressed variants
└── *.LICENSE.txt                                        # License files
```

### Data Flow

1. **Entry Point** registers custom elements, calls `Registry.initialize(hass, config)` synchronously
2. **Registry** reads entity/device/area data from `hass` object (synchronous), builds pre-computed Maps/Sets
3. **Utils** collect persons, weather, favorites using pre-filtered Registry methods
4. **Section Builders** generate overview, areas, weather/energy sections
5. **View Builders** generate utility views (lights, covers, security, batteries, climate) + per-area room views
6. **Custom Cards** render reactive UI (real-time `hass` updates via `set hass()`)

### Registry — Core Design

The Registry is a **static singleton** (no instance, all static members). Initialized once, then provides O(1) lookups everywhere.

**Synchronous Init** (reads directly from hass object, no WebSocket needed):
```
Object.values(hass.entities)  → EntityRegistryDisplayEntry[]
Object.values(hass.devices)   → DeviceRegistryEntry[]
Object.values(hass.areas)     → AreaRegistryEntry[]
```
Called once in dashboard strategy `generate()` before views are returned. Idempotent — subsequent calls in view strategies are no-ops.

**Pre-Computed Maps:**

| Map | Key | Value | Filtered? |
|-----|-----|-------|-----------|
| `_entityById` | entity_id | EntityRegistryDisplayEntry | Raw |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TheRealSimon42) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
