---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Project Overview

**Mandarin Cargo** is a Telegram Mini App and Progressive Web App (PWA) for cargo/shipping logistics management. It serves multiple user roles — end clients tracking their parcels, warehouse workers managing flights and cargo, accountants verifying transactions, and administrators managing users and roles.

The app is primarily designed to run inside Telegram via the Telegram WebApp SDK, but several internal routes (`/admin/*`, `/pos`, `/flights`, `/statistics`) are accessible directly from a browser for back-office staff.

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Framework | React 19 (with StrictMode) |
| Language | TypeScript 5.9 (strict mode, no `any`) |
| Build Tool | Vite 7 with `@vitejs/plugin-react-swc` |
| Styling | Tailwind CSS 4 + `tw-animate-css` |
| UI Primitives | Radix UI (shadcn/ui-style wrappers in `src/components/ui/`) |
| Icons | Lucide React |
| State (Server) | TanStack Query (React Query) v5 |
| State (Client) | Zustand v5 |
| Forms | React Hook Form + Zod v4 |
| Animations | Framer Motion |
| Charts | Recharts |
| Toasts | Sonner |
| Maps | Leaflet + React-Leaflet |
| QR Scanning | html5-qrcode |
| i18n | i18next + react-i18next |
| Offline DB | IndexedDB via `idb` |
| HTTP Client | Axios (two instances: JSON and multipart/form-data) |

## Build & Development Commands

```bash
npm run dev        # Start Vite dev server (host: true, port: 5173)
npm run build      # TypeScript compile (tsc -b) + Vite production build
npm run lint       # ESLint check across the project
npm run preview    # Preview production build locally
```

No test suite is configured.

## Project Structure

```
src/
  api/               # Axios clients and domain API services
    client.ts        # apiClient (JSON) & apiClientFormData (multipart)
    services/        # Per-domain async functions (auth, cargo, flights, admin, etc.)
    hooks/           # TanStack Query custom hooks (useAdminClients, etc.)
  components/        # React components
    ui/              # Shadcn/ui-style reusable primitives (button, dialog, input, etc.)
    admin/           # Admin-layout shell components
    carousel/        # Home page carousel management
    delivery/        # UzPost delivery request flow
    expectedCargo/   # Warehouse expected-cargo UI
    history/         # Client cargo history
    manager/         # Manager client data table
    modals/          # Shared modal components
    navigation/      # Nav bars (UserNav, VerificationNav, FloatingNavbar)
    notifications/   # Notification center
    pages/           # Page-level components (DeliveryHistoryPage, etc.)
    profile/         # User profile sections
    statistics/      # Charts and stat cards
    user_page/       # User home action buttons
    verification/    # Accountant verification UI
    warehouse/       # Warehouse transaction tables & filters
    wallet/          # Wallet & card management modals
  config/
    config.ts        # Runtime env var validation (VITE_API_BASE_URL, etc.)
  constants/         # Static JSON data (districts in uz/ru)
  hooks/             # Custom React hooks (useConfirm, useToast, useProfile, etc.)
  i18n/              # i18next configuration and locale files (uz.json, ru.json)
  lib/               # Utility libraries (cn, telegram helpers, validation, formatting)
  pages/             # Top-level page components
    admin/           # Admin pages (accounts, roles, audit, carousel, warehouse, etc.)
    dashboard/       # Client dashboard & track-code search
  schemas/           # Zod validation schemas
  store/             # Zustand stores (warehouse filters, expected cargo, manager state)
  types/             # Global TypeScript type definitions
    telegram-web-app.d.ts  # Custom Telegram WebApp SDK types
  utils/             # Helper modules (offlineStorage, audioUtils, numberFormat, etc.)
  App.tsx            # Root component: auth, routing, layout orchestration
  main.tsx           # React root render, QueryClient, ThemeProvider, service worker registration
  index.css          # Tailwind imports, CSS variables (light/dark), custom scrollbar, focus styles
```

## Architecture Deep Dive

### Entry & Auth Flow

1. `index.html` loads the Telegram WebApp SDK (`telegram-web-app.js`) and PWA manifest.
2. `src/main.tsx` creates the React root, wraps the tree in `QueryClientProvider` and `ThemeProvider`, and registers the service worker in production only.
3. `TelegramWebAppGuard` wraps the entire app. It:
   - Skips Telegram validation for browser routes (`/admin/*`, `/pos`, `/flights`, `/statistics`).
   - Validates `initData` with the backend for all other routes.
   - Calls `window.Telegram.WebApp.ready()` and `.expand()` on success.
   - Attempts auto-login via `/auth/telegram-login` if no token exists.
4. `App.tsx` performs the second auth gate:
   - Admin tokens live in `localStorage` (`access_token`, `admin_role`).
   - Regular user tokens live in `sessionStorage` (`access_token`).
   - Calls `/auth/me` to validate user tokens.
   - On 401, clears storage and dispatches `auth:logout`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kingprogrammer07/mandarin_web](https://github.com/Kingprogrammer07/mandarin_web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
