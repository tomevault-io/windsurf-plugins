---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project: Manager Conscious Cafe

A full-stack cafe management system with POS, KDS, stock management, and analytics.

**Stack:** React 18 (Netlify) + Supabase (PostgreSQL + Realtime + Storage)

---

## Development Commands

```bash
cd frontend
npm start        # dev server on http://localhost:3000
npm run build    # production build (output: frontend/build/)
npm test         # run tests (watch mode)
```

Deployment is via Netlify — push to `main` triggers a build automatically. Build base is `frontend/`, publish dir is `build/`, Node 20.

---

## Application Routes & Role Access

| Route | Component | Roles |
|-------|-----------|-------|
| `/pin-login` | PinLogin | public |
| `/dashboard` | Dashboard | admin, store_manager |
| `/pos` | POSPageNew | admin, cashier |
| `/kds` | KDSPage | admin, kitchen, cashier |
| `/bar-kds` | BarKDSPage | admin, bar, cashier |
| `/wood-fire-oven-kds` | WoodFireOvenKDSPage | admin, kitchen, cashier |
| `/server` | ServeOnlyPage | admin, server, cashier |
| `/table-self-orders` | TableSelfOrdersPage | admin, cashier |
| `/cloud-orders` | CloudOrdersPage | admin, cashier |
| `/manager` | RecipeManager | admin |
| `/create` | RecipeForm | admin |
| `/ingredients` | IngredientsManager | admin, store_manager |
| `/stock` | StockRegister | admin, store_manager |
| `/analytics` | Analytics | admin, cashier |
| `/data` | DataManager | admin, cashier |
| `/settings` | SettingsPage | admin |
| `/invoice/:orderId` | InvoicePage | public (customer WhatsApp link) |
| `/self-order` | SelfOrderPage | public (table QR scan) |

All heavy components are **lazy-loaded** via `React.lazy()`. CSS for POS, DataManager, RecipeManager is imported eagerly in `App.js` to prevent flash on first navigation.

`recipes` and `ingredients` are lazy-fetched in `App.js` — only loaded on first visit to `/manager`, `/create`, `/analytics`, or `/dashboard`. **POS never uses these** — it calls `getPOSRecipes()` which is a lightweight query without ingredient joins.

---

## Key Entry Points

| Feature | File |
|---------|------|
| App router + lazy loading | `frontend/src/App.js` |
| POS root | `frontend/src/components/POS/POSPageNew.js` |
| POS global state | `frontend/src/components/POS/context/POSContext.js` |
| Cart + billing UI | `frontend/src/components/POS/layout/RightCartPanel.js` |
| Table grid | `frontend/src/components/POS/views/TablesFullPageView.js` |
| Split bill | `frontend/src/components/POS/views/SplitBillView.js` |
| KDS | `frontend/src/components/KDS/KitchenDisplay.js` |
| Analytics (Sales Analysis) | `frontend/src/components/Analytics.js` |
| Order data page | `frontend/src/components/DataManager.js` |
| Order service | `frontend/src/services/orderService.js` |
| Table service | `frontend/src/services/tableService.js` |
| Payment service | `frontend/src/services/paymentService.js` |
| Recipe/ingredient CRUD | `frontend/src/services/supabaseService.js` |
| POS styles | `frontend/src/styles/POSNew.css` |

---

## Authentication

PIN-based — no Supabase Auth. Staff enter a 4-digit PIN which is verified server-side via the `verify_pin` RPC (bcrypt comparison in Postgres). The returned user profile is stored in `sessionStorage`.

- `authService.getCurrentUser()` — reads from sessionStorage
- `authService.hasRole('admin')` — checks current user's role
- `RoleBasedRoute` wraps all protected routes; unauthenticated → `/pin-login`
- After login, each role redirects to its default: `admin`→`/dashboard`, `cashier`→`/pos`, `kitchen`→`/kds`, `bar`→`/bar-kds`, `server`→`/server`, `store_manager`→`/dashboard`

---

## Dine-In Order State Machine

```
TABLE: available
  │ Staff taps table card in TablesFullPageView
  ▼
CONTEXT: selectedTable set, currentOrder fetched (or null if new)
  │ Staff adds items to cart → clicks "Send to Kitchen"
  ▼
TABLE: occupied  |  ORDER: status=pending, payment_status=unpaid
  │ KDS marks items preparing/ready/served  [auto-drives order.status]
  ▼
ORDER: status= cooking → ready → served
  │ Staff clicks "Generate Bill" in RightCartPanel
  │   → tableService.updateTableStatus(id, 'billed')
  │   → setSelectedTable({...table, status:'billed'})
  │   → print window opens
  ▼
TABLE: billed  |  ORDER: payment_status=unpaid
  [RightCartPanel shows: Pay Bill | Reprint | Reopen Table | Split Bill]

  ─── Reopen path ───────────────────────────────────────────────
  │ Staff clicks "Reopen Table"
  │   → tableService.updateTableStatus(id, 'occupied')
  │   → setSelectedTable({...table, status:'occupied'})
  └── Back to ORDER ACTIVE

  ─── Payment path ──────────────────────────────────────────────
  │ Staff clicks "Pay Bill" → selects method → "Process Payment"
  │   → paymentService.createPayment()  → DB: payment_status=paid
  │   → orderService.updateOrderStatus(id, 'completed')
  │   → orderService.getOrderById(id)   → setCurrentOrder(updated)
  ▼
ORDER: status=completed, payment_status=paid
TABLE: still billed  [Release Table button now visible]
  │ Staff clicks "Release Table"
  │   → tableService.clearTable(id)     → DB: table.status=available

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bhavnindersingh/RecipeManager](https://github.com/bhavnindersingh/RecipeManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
