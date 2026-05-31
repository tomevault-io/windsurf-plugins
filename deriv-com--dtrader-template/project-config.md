---
trigger: always_on
description: Enables clean imports: `import App from 'App/index'` instead of `../../../App`
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

# Derivatives Trading Platform - Architecture Guide

A comprehensive TypeScript/React-based derivatives trading platform built with MobX state management, React Router navigation, and WebSocket-based real-time API integration.

## Project Overview

**Repository:** Monorepo with 9 NPM packages using npm workspaces
**Main Tech Stack:** React 18, MobX 6, TypeScript 5, Webpack 5, Jest 29
**Build System:** Webpack with separate configs per package
**Architecture Pattern:** Multi-store reactive state management + component-driven UI
**Node Version:** 20.x (required — no `.nvmrc`, manage manually)

## Common Commands

### Development

```bash
# Install dependencies
npm run bootstrap

# Start development server for core package
npm run serve core
# Runs at https://localhost:8443

# Start specific package
npm run serve trader
npm run serve reports

# Build all packages
npm run build:all

# Build specific package
npm run build --workspace=@deriv/core
```

### Testing

```bash
# Run all tests (stylelint + eslint + jest)
npm run test

# Run only Jest tests
npm run test:jest

# Run single test file
npm run test:jest -- path/to/test.spec.ts

# Run ESLint on all packages
npm run test:eslint-all

# Run Stylelint on all SCSS files
npm run test:stylelint

# Fix style issues automatically
npm run stylelint:fix

# Format code with Prettier
npm run prettify
```

### Utilities

```bash
# Generate color tokens (run after editing brand.config.json colors)
npm run generate:colors

# Validate white-label configuration
npm run verify:whitelabel

# Clean all node_modules (interactive)
npm run clean
```

---

## White-Label Configuration

This is a white-label template. All branding is driven by **`brand.config.json`** in the repo root — no source code changes needed for rebranding.

### White-label workflow

1. Edit `brand.config.json` (brand name, colors, API endpoints, hostnames, app IDs)
2. Replace SVG logo files in `assets/brand/` (`brand-logo.svg`, `brand-logo-dark.svg`, `platform-logo.svg`)
3. Run `npm run generate:colors` — regenerates SCSS tokens in `packages/shared/src/styles/`
4. Run `npm run verify:whitelabel` — validates config and logo files
5. Run `npm run build:all`

**Never edit the generated SCSS files directly** — they are overwritten by `generate:colors`:

- `packages/shared/src/styles/constants/colors.scss`
- `packages/shared/src/styles/tokens/brand.scss`
- `packages/shared/src/styles/tokens/semantic.scss`
- `packages/shared/src/styles/tokens/components.scss`

### Key config fields

| Field                               | Purpose                                                                                                        |
| ----------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| `brand_domain`                      | Single production domain string (e.g. `"yourdomain.com"`). Used for environment detection and security checks. |
| `brand_hostname.staging/production` | Platform hostname for each environment. Also used as the canonical URL and OAuth redirect URI base.            |
| `platform.home_url`                 | URL the sidebar Home button navigates to.                                                                      |
| `platform.help_centre_url`          | URL the sidebar Help button opens.                                                                             |
| `signup_url.staging/production`     | Signup page URLs — used by the signup button when enabled.                                                     |
| `derivws.staging/production`        | DerivWS WebSocket API base URL.                                                                                |
| `features.dark_mode`                | Show/hide the dark mode toggle in the sidebar and mobile menu. Default: `false`.                               |
| `features.language_switcher`        | Show/hide the language switcher in the sidebar and mobile menu. Default: `false`.                              |

### App ID

The platform connects to the Deriv WebSocket API (v3). Register your own App ID at [https://developers.deriv.com](https://developers.deriv.com) and set it in `brand.config.json` under `app_id.staging` and `app_id.production`.

For full details, see [WHITE_LABEL.md](docs/WHITE_LABEL.md).

---

## 1. State Management Architecture (MobX Stores)

### Overall Pattern

The platform uses a **root store + modular stores** architecture with MobX as the state management library:

- **Root Store** (`RootStore`): Central state container instantiated once per app
- **Base Store**: Abstract parent class for all stores providing persistence and lifecycle management
- **Modular Stores**: Specialized stores for different domains (Trading, Portfolio, Contracts, etc.)

### Key Stores in Core Package (`packages/core/src/Stores/`)

```
RootStore (root)
├── client: ClientStore          # Auth, user profile, accounts
├── common: CommonStore          # Platform config, language, theme
├── ui: UIStore                  # Modal states, notifications, UI toggles

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deriv-com/dtrader-template](https://github.com/deriv-com/dtrader-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
