---
trigger: always_on
description: DeclaRenta converts foreign broker reports into Spanish tax declarations (Modelo 100, 720, 721, D-6). Browser-first, privacy-focused. All financial data stays on the user's machine.
---

# CLAUDE.md - DeclaRenta

## Project Overview

DeclaRenta converts foreign broker reports into Spanish tax declarations (Modelo 100, 720, 721, D-6). Browser-first, privacy-focused. All financial data stays on the user's machine.

- **Domain**: [declarenta.com](https://declarenta.com)
- **Alt URL**: [geiserx.github.io/DeclaRenta](https://geiserx.github.io/DeclaRenta/)
- **Docker**: `drumsergio/declarenta` on Docker Hub

### Supported Brokers (10)

IBKR (XML), Degiro (CSV), eToro (XLSX), Scalable Capital (CSV), Freedom24 (JSON), Revolut (XLSX), Lightyear (CSV), Coinbase (CSV), Binance (CSV), Kraken (CSV)

## Architecture

```
src/
  types/         TypeScript interfaces (broker, tax, ECB, IBKR)
  parsers/       Broker-specific parsers (10 brokers + auto-detect)
    index.ts     detectBroker() auto-detection, brokerParsers registry
    ibkr.ts      IBKR Flex Query XML
    degiro.ts    Degiro CSV (auto-detect delimiter)
    etoro.ts     eToro XLSX (6+ header variants)
    scalable.ts  Scalable Capital CSV
    freedom24.ts Freedom24 JSON
    revolut.ts   Revolut XLSX (Trading Account Statement)
    lightyear.ts Lightyear CSV (Transaction Report)
    coinbase.ts  Coinbase CSV
    binance.ts   Binance CSV
    kraken.ts    Kraken CSV
  engine/        Core calculation modules
    fifo.ts      FIFO cost basis engine (Art. 37.2 LIRPF)
    ecb.ts       ECB exchange rate fetcher (SDMX API)
    wash-sale.ts Anti-churning rule detector (Art. 33.5.f LIRPF, 2 months listed / 1 year unlisted)
    dividends.ts Dividend + withholding tax processor
    double-taxation.ts  Double taxation deduction (Art. 80 LIRPF, Casilla 0588)
    dates.ts     Date normalization utilities
  generators/    Output generators
    report.ts    Modelo 100 casilla mapper
    modelo720.ts AEAT 720 fixed-width file (500 bytes/record, ISO-8859-15)
    modelo721.ts Modelo 721 stub (real format is XML per Orden HFP/886/2023)
    d6.ts        D-6 report generator (AFORIX format)
    csv.ts       CSV export
  cli/           CLI entry point (commander)
  web/           Browser UI (Vite, vanilla TS)
    main.ts        Entry point, splash screen, wizard orchestration, file upload
    sidebar.ts     Hash-based routing (#perfil, #renta, #m720, #d6), mobile toggle
    profile.ts     Fiscal profile form (NIF, name, CCAA, phone, year → localStorage)
    broker-guides.ts  Visual broker card grid + step-by-step download guides
    section-720.ts    Modelo 720 section (threshold bar, positions, filing guide)
    section-721.ts    Modelo 721 section (crypto threshold, positions, filing guide)
    section-d6.ts     Modelo D-6 section (positions, AFORIX guide, copy-to-clipboard)
    wizard.ts      3-step wizard within Renta section (Upload → Review → Results)
    charts.ts      Donut, bar, monthly G/L charts (pure SVG, no libs)
    casilla-detail.ts  Expandable casilla cards with legal references
    year-compare.ts    Year-over-year comparison (localStorage persistence)
    disclaimer.ts  Legal disclaimer modal
    style.css      Full CSS (dark/light themes, sidebar layout, splash, responsive)
  i18n/          Internationalization
    index.ts     t() function, locale management, localechange event
    locales/     es.ts, en.ts, ca.ts, eu.ts, gl.ts (5 languages)
tests/           Vitest tests mirroring src/ structure
```

## Tech Stack

- **Language**: TypeScript (strict mode, ES2022)
- **XML parsing**: `fast-xml-parser` (IBKR Flex Query XML)
- **XLSX parsing**: `xlsx` (eToro)
- **Decimal math**: `decimal.js` (financial precision, never raw Number)
- **CLI**: `commander`
- **Build**: `tsup` (library/CLI), `vite` (web)
- **Test**: `vitest`
- **CI**: GitHub Actions (Node 20 + 22)
- **Docker**: Multi-stage Dockerfile.web (node build → nginx:1.27-alpine)

## Web UI Architecture

### Layout
- **Sidebar + content area** grid layout (`grid-template-columns: 260px 1fr`)
- 5 sections: Perfil fiscal, Modelo 100 (Renta), Modelo 720, Modelo 721, Modelo D-6
- Hash-based routing (`location.hash`): `#perfil`, `#renta`, `#m720`, `#m721`, `#d6`
- Mobile (≤768px): sidebar collapses, hamburger toggle with backdrop

### Splash Screen
- Full-screen landing shown on every page load (no localStorage skip)
- Positioned below top-bar (`top: var(--topbar-h)`) so language/theme toggles stay accessible
- **GOTCHA**: `.splash { display: flex }` overrides the `hidden` attribute. Use `splash.style.display = "none"` (inline style), never `splash.hidden = true`
- Logo click in top-bar returns to splash via `showSplash()`

### i18n System
- 5 locales: es, en, ca, eu, gl
- Static text: `data-i18n` attributes updated by `updateStaticText()`
- Dynamic content (broker guides, profile form, 720/D-6 sections): rendered with `t()` calls, must re-render on `localechange` event
- **GOTCHA**: Any module that renders HTML with `t()` must listen for `localechange` and re-render, otherwise switching language leaves stale text

### Data Persistence
- **localStorage only** — no cookies, no server-side storage
- `declarenta_profile`: fiscal profile (NIF, name, CCAA, phone, year)
- `declarenta_reports_*`: saved reports for year comparison

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GeiserX/DeclaRenta](https://github.com/GeiserX/DeclaRenta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
