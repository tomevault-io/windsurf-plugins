---
trigger: always_on
description: **irDashies** is an Electron-based iRacing overlay/dashboard app using React, TypeScript, Zustand, and Tailwind CSS.
---

# irDashies - Development Guide

## Project Overview

**irDashies** is an Electron-based iRacing overlay/dashboard app using React, TypeScript, Zustand, and Tailwind CSS.

- **Repo**: https://github.com/tariknz/irdashies (fork)
- **Branch**: `main` | **License**: MIT

---

## Architecture Rules (read first)

Before making any non-trivial change, read [`docs/ARCHITECTURE_RULES.md`](docs/ARCHITECTURE_RULES.md). It contains the hard rules for layering, telemetry subscriptions, stores, IPC/bridges, processors, storage, widgets, settings migration, native code, logging, security, performance, and testing — plus a Pre-PR checklist.

The accompanying [`docs/ARCHITECTURE_REVIEW.md`](docs/ARCHITECTURE_REVIEW.md) explains the _why_: findings from the architecture audit, the target topology, and the phased implementation plan. Reference the relevant phase in your PR description.

**When `ARCHITECTURE_RULES.md` conflicts with anything in this file, the rules file wins for architecture concerns.**

---

## Git Workflow

**Never commit directly to `main`** - always use feature branches and PRs.

### Branch Naming

- `feat/feature-name` | `fix/bug-description` | `chore/task` | `wip/experimental`

### Commit Format

```
<type>: <description>
# Examples: feat: add avg lap time column | fix: resolve linting errors
```

### Clean Feature Branches (cherry-pick)

```bash
git checkout main && git checkout -b feat/feature-clean
git cherry-pick <hash>              # Full commit
git cherry-pick -n <hash>           # Partial: then reset/checkout unwanted files
```

### Pull Requests

When opening a PR, follow `.github/pull_request_template.md` exactly. Fill in the **Description** and **Screenshots** (Before / After) sections, tick the matching **Type of Change** box, and only check **Checklist** items you've actually done — don't tick iRacing testing if you only verified in Storybook.

---

## Project Structure

```
src/
├── app/                  # Electron main process
│   ├── bridge/           # IPC bridges (dashboard, iracingSdk)
│   ├── irsdk/            # iRacing SDK wrapper
│   └── storage/          # Dashboard persistence
├── frontend/             # React UI
│   ├── components/       # Widgets (Standings, Input, Settings, etc.)
│   ├── context/          # Zustand stores & React providers
│   └── utils/            # Shared utilities (colors, time)
└── types/                # Shared TypeScript types, widget configs, default dashboard
```

---

## State Management

| Use Case                                           | Solution                         |
| -------------------------------------------------- | -------------------------------- |
| Cross-component, high-frequency (60 FPS telemetry) | **Zustand** with custom equality |
| UI-only, single component                          | **useState**                     |
| Configuration, callbacks, infrequent updates       | **React Context**                |

### Core Stores

```typescript
// Telemetry (real-time)
const speed = useTelemetry('Speed', customComparator);
const playerSpeed = useTelemetryValue('Speed');

// Session (drivers, positions)
const drivers = useSessionDrivers();
const playerCarIdx = useDriverCarIdx();
```

### Telemetry Precision — `useTelemetryValuesRounded`

`useTelemetryValues` uses exact equality (`===`). Continuous float arrays like `CarIdxLapDistPct` change on every 60fps tick, causing subscriptions to fire every frame even when the UI doesn't need to update.

Use `useTelemetryValuesRounded(key, precision)` for float arrays where sub-threshold changes produce no visible or meaningful difference:

```typescript
// Position on track — 3dp = ~0.5m resolution on a 5km track
const lapDistPcts = useTelemetryValuesRounded('CarIdxLapDistPct', 3);

// Time-based gaps displayed to 0.1s — 2dp = 10ms resolution
const estTimes = useTelemetryValuesRounded('CarIdxEstTime', 2);
```

**Precision guidelines by use case:**

| Use Case                                         | Hook                        | Precision | Rationale                                                                                |
| ------------------------------------------------ | --------------------------- | --------- | ---------------------------------------------------------------------------------------- |
| Track map position                               | `useTelemetryValuesRounded` | 3dp       | 0.1% of track length (~22m on Nords, ~5m on a 5km track) — sub-pixel on any rendered map |
| Position sorting (standings)                     | `useTelemetryValuesRounded` | 3dp       | Same physical resolution; sufficient to distinguish order                                |
| Time delta display (0.1s)                        | `useTelemetryValuesRounded` | 2dp       | 10ms resolution, display is 100ms                                                        |
| Time interpolation (reference lap)               | `useTelemetryValuesRounded` | 4dp       | REFERENCE_INTERVAL = 0.0025; 4dp keeps error <10ms                                       |
| Smooth animation (throttle, steering)            | `useTelemetryValues`        | —         | Full precision needed for 60fps feel                                                     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tariknz/irdashies](https://github.com/tariknz/irdashies) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
