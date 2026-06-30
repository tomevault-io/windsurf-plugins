---
trigger: always_on
description: Full-stack DIY weather monitoring system that collects real environmental data from Arduino/ESP32 hardware and exposes it via a PHP REST API with a Next.js web dashboard.
---

# Arduino Weather Station — Claude Code Workspace

## Project Overview

Full-stack DIY weather monitoring system that collects real environmental data from Arduino/ESP32 hardware and exposes it via a PHP REST API with a Next.js web dashboard.

- **Live demo:** https://meteo.miksoft.pro
- **License:** MIT
- **Default locale:** Russian (`ru`), also supports English (`en`)

---

## Tech Stack

| Layer | Technology |
|---|---|
| Firmware | C++ (Arduino / ESP32), Arduino IDE |
| Backend | PHP 8.2+, CodeIgniter 4.6, MySQL |
| Frontend | Next.js 16, React 19, TypeScript 5.9 |
| State management | Redux Toolkit + RTK Query, next-redux-wrapper |
| Styling | SASS/SCSS, CSS Modules, next-themes |
| Charts | ECharts 6 (echarts-for-react) |
| i18n | i18next, next-i18next (locales: `ru`, `en`) |
| HTTP client | Guzzle 7 (PHP), RTK Query (TS) |
| Package manager | Yarn 4 (client), Composer (server) |
| Testing | PHPUnit 11.5 (server), Jest 30 + Testing Library (client) |
| Quality | SonarCloud, ESLint, Prettier |
| CI/CD | GitHub Actions → FTP deploy |

---

## Team Agents

| Agent | Code Location                                 | Instructions               |
|-------|-----------------------------------------------|----------------------------|
| **Backend Agent** | `/server`                                 | `/.claude/agents/backend.md` |
| **Frontend Agent** | `/client`                                | `/.claude/agents/frontend.md` |
| **QA Agent** | `/server/tests`, `/client/**/*.tests.(ts/tsx)` | `/.claude/agents/qa.md`       |
| **Doc Agent** | `/docs`, `README.md`                          | `/.claude/agents/docs.md`  |

Agents must read their instruction file before starting. Each agent reports completion to Team Lead.

---

## High-Level Architecture

```
Arduino/ESP32 Hardware
  │  POST /sensors (every 60 s)
  ▼
CodeIgniter 4 REST API  ──► MySQL Database
  │                              ▲
  │  CLI cron commands           │
  └─► External Weather APIs ─────┘
        (OpenWeatherMap, WeatherAPI, VisualCrossing)
        (NarodMon — data push)

Next.js Frontend (SSR)
  └─► RTK Query ──► PHP API
```

### Data Flow
1. Arduino POSTs sensor readings to `POST /sensors`
2. CLI commands (`spark system:getCurrentWeather` etc.) periodically fetch from external APIs and compute hourly/daily averages
3. Next.js pages fetch data via RTK Query endpoints; locale is sent as a custom `Locale` HTTP header
4. The `System` controller also pushes data to narodmon.ru

---

## Directory Structure

```
arduino-weather-station/
├── arduino/               # Firmware
│   ├── main/              # Production sketch (main.ino + sensor modules)
│   ├── i2c_scanner/       # I2C device discovery utility
│   └── test_*/            # Individual sensor test sketches
├── client/                # Next.js frontend
│   ├── api/               # RTK Query store, slices, endpoint definitions, types
│   ├── components/        # Reusable React components (widgets, layout, icons)
│   ├── pages/             # Next.js pages (index, climate, sensors, history, heatmap, forecast)
│   ├── public/locales/    # i18n translation files (en/, ru/)
│   ├── styles/            # Global SASS + light/dark theme CSS
│   ├── tools/             # Utility functions, custom hooks, unit tests
│   └── ui/                # Small UI primitives (theme-switcher, comparison-icon)
├── server/                # CodeIgniter 4 backend
│   ├── app/
│   │   ├── Controllers/   # REST controllers (Current, Forecast, Heatmap, History, Sensors, System)
│   │   ├── Database/      # Migrations + Seeds
│   │   ├── Entities/      # Data entities (WeatherData, WeatherDataEntity, WeatherForecastEntity)
│   │   ├── Libraries/     # External API clients (OpenWeather, WeatherAPI, VisualCrossing, NarodMon)
│   │   ├── Models/        # DB models (RawWeatherDataModel, Hourly/DailyAveragesModel, ForecastWeatherDataModel)
│   │   └── Config/        # Routes.php, app configuration
│   └── tests/             # PHPUnit test suites (unit/, database/, session/)
├── models/                # 3D-printable STL/CAD files for enclosure
├── docs/                  # Documentation assets and screenshots
├── config/                # Shared configuration files
└── .github/workflows/     # GitHub Actions (sonarcloud, ui-checks, ui-deploy, api-deploy, arduino-code-check)
```

---

## Development Commands

### Client (Next.js) — run from `client/`

```bash
yarn install              # Install dependencies
yarn dev                  # Start dev server (http://localhost:3000)
yarn build                # Production build
yarn start                # Start production server
yarn test                 # Run Jest unit tests
yarn test:coverage        # Run tests with coverage report
yarn eslint:check         # ESLint check
yarn eslint:fix           # ESLint auto-fix
yarn prettier:check       # Prettier check
yarn prettier:fix         # Prettier auto-fix
```

### Server (PHP) — run from `server/`

```bash
composer install                    # Install PHP dependencies
php spark migrate                   # Run database migrations
php spark db:seed                   # Run database seeds

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [miksrv/arduino-weather-station](https://github.com/miksrv/arduino-weather-station) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
