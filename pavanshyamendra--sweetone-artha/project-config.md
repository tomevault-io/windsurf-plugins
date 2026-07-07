---
trigger: always_on
description: Electron desktop app for **branch billing staff** at Sweetone. Handles POS billing, stock management, distribution acceptance, and bulk order viewing. All data syncs to/from **sweetone-oms**, which is the single backend.
---

# sweetone-artha

Electron desktop app for **branch billing staff** at Sweetone. Handles POS billing, stock management, distribution acceptance, and bulk order viewing. All data syncs to/from **sweetone-oms**, which is the single backend.

This app is distinct from **sweetone-retail** (which handles distribution *creation* and dispatch). Artha handles the *receiving* side at each branch.

## Related Projects

**sweetone-oms** lives at `~/Documents/sweetone-oms` and hosts the entire API.
- Deployed at `https://oms.sweetone.in`
- Next.js App Router with Prisma + PostgreSQL
- **When changing any API call in this project, also update the corresponding route handler in sweetone-oms (`src/app/api/...`).**
- **When adding a new field to a Bill, StockLive entry, or Distribution, update the Prisma schema in sweetone-oms and the TypeScript types in both projects.**

**sweetone-retail** lives at `~/Documents/sweetone-retail` — handles distribution *creation* and dispatch (the sending side). Artha handles *acceptance*.

## API Integration

Base URL: `src/electron/config.ts` → `BACKEND_URL = "https://oms.sweetone.in"`

All requests use `Authorization: Bearer <accessToken>` from `storageService.getToken()`.

### Auth — `src/electron/services/auth.service.ts`
| Function | OMS route | Notes |
|---|---|---|
| `getLogin(email, password)` | `POST /api/auth/login` | Returns `accessToken`, `refreshToken`, user info; stored in electron-store |
| `refreshToken()` | `POST /api/auth/refresh` | Sends `refreshToken` as `Cookie` header |

User fields stored in electron-store after login: `authToken`, `refreshToken`, `userId`, `userEmail`, `userName`, `userRole`, `userBranchId`, `userBranchName`.

### Billing — `src/electron/services/billing.service.ts`
| IPC channel | Method + OMS route | Notes |
|---|---|---|
| `save-order` | Local SQLite only (then auto-sync) | Saves bill locally, prints immediately; sync happens separately |
| `sync-orders` | `POST /api/bill/sync` | Body: `{ orders: unsyncedOrders }`. Returns `{ syncedIds }`. Marks synced in local DB. |
| `stocklive-sync` | `GET /api/stocklive` | Returns live stock levels for the branch |
| `update-stock` | `POST /api/bill/stocklive` | Body: `{ branchId, menuItemId, quantity, type: "add"\|"deduct"\|"set" }` |
| `print-stocklive` | Local print only | No API call; generates PDF via `createStockLivePDF` |

### Menu & Stock — `src/electron/services/menu.service.ts`
| IPC channel / Function | Method + OMS route | Notes |
|---|---|---|
| `get-menu-items` / `fetchMenuItems()` | `GET /api/menu` | All menu items; cached in electron-store |
| `sync-data-menu` / `syncDataMenu()` | `GET /api/branch` + `GET /api/menu/:branchId` | Parallel; also calls `syncOrdersDb()` first. Branch-specific pricing. |
| `transfer-stock` / `transferStock()` | `POST /api/stocklive/transfer` | Body: `{ fromBranchId, toBranchId, menuItemId, quantity }`. Prints transfer PDF on success. |
| `return-stock` / `returnStock()` | `POST /api/stocklive/returnStock` | Body: `{ branchId, menuItemId, quantity }`. Prints return PDF on success. |
| `get-menu-item-branchwise` | Local electron-store cache | Returns cached `menuItemsBranchwise_<branchId>` |

### Distributions — `src/electron/ipc/distribution.handlers.ts`
| IPC channel | Method + OMS route | Notes |
|---|---|---|
| `get-distributions` | `GET /api/distribute?date=` | Lists distributions for a given date |
| `get-distribution` | `GET /api/distribute/:id` | Single distribution |
| `get-current-distribution` | `GET /api/distribute/:id` | Same route, used from different UI flow |
| `get-branches` | `GET /api/branch` | Cached in electron-store |
| `create-distribution` | `POST /api/distribute` | Used for creating (less common from artha) |
| `put-distribution` | `PUT /api/distribute` | Body: updated distribution data |
| `accept-distribution` | `POST /api/distribute/accept` | Body: `{ distributionId, branchId, acceptedByUserId?, items: [{ distributionItemId, receivedQuantity?, receivedTrayWeight?, missingQuantity?, note? }] }` |

### Bulk Orders — `src/electron/services/bulkOrders.service.ts`
| IPC channel | Method + OMS route | Notes |
|---|---|---|
| `get-orders` | `GET /api/orders?date=&status=` | View-only; branch staff monitors incoming orders |

### Realtime Online-Order Printing (Pusher)
Incoming online orders (Rista webhook → OMS) are pushed to the branch app and auto-printed, instead of polling. Vercel serverless can't hold WebSockets, so Pusher sits between OMS and artha.
- **OMS**: `src/app/api/rista/webhooks/route.ts` → on `order.status = "Created"`, resolves branchCode→branchId via `resolveBranchIdFromCode` (`src/lib/rista/branch-map.ts`, reverse of the `rista_branch_map` setting) and calls `triggerBranchEvent` (`src/lib/pusher/server.ts`) with the full Rista sale detail on channel `branch-<branchId>`, event `rista-order.created`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PavanShyamendra/sweetone-artha](https://github.com/PavanShyamendra/sweetone-artha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
