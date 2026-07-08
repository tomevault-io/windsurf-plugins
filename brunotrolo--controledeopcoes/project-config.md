---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ControleDeOpcoes** is a Google Apps Script (GAS) web application for managing a Brazilian stock options portfolio (B3 exchange). It consists of:
- A **GAS backend** (`.gs` files) that reads/writes Google Sheets and calls external APIs
- A **Vue 3 frontend** (`.html` files) assembled at runtime by GAS's `include()` template system

There is no build step, no `package.json`, and no local dev server. All execution happens inside Google's Apps Script runtime.

## Deployment Workflow

### Automatic (CI/CD) — preferred

**Every push to `main` automatically deploys to the GAS DEV project** via GitHub Actions (`.github/workflows/deploy-gas-dev.yml`). The job runs `clasp push --force`, then updates the web app deployment (reusing the existing one — GAS caps scripts at 20 versioned deployments) and reads the **real web app URLs from the Apps Script API** (`entryPoints[].webApp.url`); they appear in the step summary and are committed to `.webapp-urls` (key `HEAD_URL` = always-latest code). Never hand-construct `/macros/s/<id>/dev` URLs. Manual trigger: **Actions → Deploy to GAS DEV → Run workflow**.

Full setup guide (reusable for other projects): `docs/GUIA_CICD_GITHUB_GAS.md`.

Key files:
- `.clasp.json` — points to the DEV scriptId (committed; scriptId is not a secret)
- `.claspignore` — excludes non-GAS files (`pwa-mobile/`, `mockups/`, `docs/`, `.github/`, `*.md`). **Any new non-GAS file/folder must be added here**, otherwise GAS tries to execute it and crashes (e.g. a PWA `sw.js` caused `ReferenceError: self is not defined`)
- GitHub Secret `CLASPRC_JSON` — clasp OAuth credentials (classic `"token"` + `"oauth2ClientSettings"` format, **not** the clasp v3 `"tokens"` format)

### Manual (fallback)

```bash
# Push local changes to GAS
clasp push

# Open the GAS editor
clasp open

# Deploy as web app (done from the GAS IDE: Deploy > Manage Deployments)
```

Running individual backend functions (tests, syncs) is done by selecting the function name in the GAS editor and clicking **Run**.

## Running Tests

Each `.gs` module has a dedicated integration test function. Run them from the GAS editor by selecting the function and clicking Run:

| File | Test Function |
|---|---|
| `000_CoreServiceAPIClient.gs` | `testSuiteApiClient()` |
| `001_CoreServiceConfig.gs` | `testConfigArchitectureV5()` |
| `002_CoreDataUtils.gs` | `testSuiteDataUtilsV2()` |
| `004_CoreServiceLogger.gs` | `testSuiteLoggerV3()` |
| `005_CoreServiceUI.gs` | `testSuiteUIHandler()` |
| `006_CoreOrchestrator.gs` | `testSuiteOrchestrator()` |

To validate the full infrastructure stack at once, run `testeFinalIntegridade()` in `Código.gs`.

## Required Script Properties

Set these in **GAS Project Settings > Script Properties**:

- `OPLAB_ACCESS_TOKEN` — OPLab API authentication token
- Claude API key — configured via `025_ConsultorIAClaudeSonnet45.gs`

## Architecture

### Backend Layer (`.gs` files — numbered by load order)

Files are numbered `000`–`025`; GAS loads them in alphabetical order, so numbering enforces dependency order.

**Infrastructure (000–005) — must be loaded before any engine:**

| File | Singleton | Responsibility |
|---|---|---|
| `000_CoreServiceAPIClient.gs` | `ApiClient`, `OplabService` | HTTP fetch with retry/backoff; OPLab API adapter |
| `001_CoreServiceConfig.gs` | `SYS_CONFIG`, `ConfigManager` | Sheet name map, Universal Data Dictionary (DUD), 3-layer config cache |
| `002_CoreDataUtils.gs` | `DataUtils` | Header maps, row maps, merge helpers, date/float parsing |
| `003_CoreSanitizador.gs` | `Sanitizador` | Input sanitization (BRL currency, pt-BR dates, tickers) |
| `004_CoreServiceLogger.gs` | `SysLogger`, `DataExtractorService` | Buffer-then-flush logging to LOGS sheet; cockpit/asset extractors |
| `005_CoreServiceUI.gs` | `UIHandler` | Silent bridge pattern for all menu-triggered functions |

**Orchestration:**

- `006_CoreOrchestrator.gs` — `CoreOrchestrator`: reads `Orquestrador_Sequencia_Padrao` and `Orquestrador_Sequencia_OPLab` keys from `Config_Global` sheet (semicolon-separated function names) and executes them sequentially
- `Código.gs` — Entry points: `onOpen()` (builds menu), `doGet()` (serves the web app), `include()` (template slot system)

**Sync Engines (007–022):**

- `007_CoreUpdatePortfolio.gs` — syncs NECTON_IMPORT → COCKPIT
- `008–009` — stock price data and 250-day history
- `010–011` — options data and options history
- `012–013` — Greeks (via OPLab API and native Black-Scholes)
- `014–022` — OPLab market data (series, best rates, volumes, variations, rankings, correlations, fundamentals)

**AI Module (025):**

- `025_ConsultorIAClaudeSonnet45.gs` — Claude Sonnet 4.5 portfolio advisor; writes history to `CONSULTOR_IA_HISTORICO`

**Other backends:**

- `API.gs` — `getDadosLight()` (fast initial load: COCKPIT + CONFIG) and `getAbasPesadas()` (background load of all 20+ sheets), plus all `google.script.run`-callable functions
- `OpLabExplorer_API.gs` — direct OPLab API passthrough for the explorer panel

### Frontend Layer (`.html` files — Vue 3, no build step)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brunotrolo/ControleDeOpcoes](https://github.com/brunotrolo/ControleDeOpcoes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
