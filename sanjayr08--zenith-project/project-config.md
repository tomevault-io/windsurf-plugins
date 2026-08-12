---
trigger: always_on
description: **Zenith** is a full-stack, AI-driven pharmacy inventory management system combining:
---

# Zenith Pharmacy Inventory System - AI Agent Instructions

## Project Overview
**Zenith** is a full-stack, AI-driven pharmacy inventory management system combining:
- **Backend**: Node.js/Express (TypeScript) with PostgreSQL + Prisma ORM
- **Frontend**: React 19 + Vite + Tailwind CSS
- **AI Service**: Python FastAPI with LangChain + Anthropic Claude
- **Payment**: PayPal subscription integration
- **Real-time**: Socket.IO for live updates

**Core Value**: Implements strict FEFO (First-Expiry-First-Out) inventory tracking with AI-powered analytics, expiry prediction, and demand forecasting.

---

## Architecture Essentials

### Three-Tier Backend Design
1. **Routes** (`src/routes/*.ts`): Define endpoints, validate input
2. **Controllers** (`src/controllers/*.ts`): Handle requests, orchestrate services
3. **Services** (`src/services/*.ts`): Business logic, database transactions, external APIs

Example flow: `POST /api/v1/sales` → `sales.controller.ts` → `SalesService.ts` computes FEFO logic → `InventoryService.ts` deducts batch.

### FEFO (First-Expiry-First-Out) Engine
**Critical pattern in this codebase.** When processing a sale:
1. `SalesService.processSale()` retrieves all available batches for a drug
2. Sorts by `expiryDate` ascending
3. Deducts quantity from earliest expiring batch first
4. Creates `InventoryMovement` records for audit trail

See: `src/services/InventoryService.ts:35-80` (batch retrieval with expiry sorting)

### Database Schema (Prisma)
**Key models for FEFO**:
- `Drug`: Product catalog
- `PurchaseBatch`: Inventory batch with `expiryDate`, `quantityRemaining`, `receivedAt`
- `Sale` + `SaleItem`: Transaction records
- `InventoryMovement`: Immutable audit trail (PURCHASE, SALE, ADJUSTMENT)

**Critical**: Always use Prisma transactions for atomic multi-model operations (e.g., sale + movement + batch update).

See: `prisma/schema.prisma` for full schema; `src/services/SalesService.ts:45-100` for transaction example.

---

## Core Patterns & Conventions

### 1. Error Handling
Use custom `AppError` (extends Error) with `statusCode` and `message`:
```typescript
// src/utils/AppError.ts
class AppError extends Error {
  statusCode: number;
  constructor(message: string, statusCode: number) { ... }
}

// Usage in controllers (wrapped with catchAsync):
if (!inventory) throw new AppError("Not found", 404);
```
**All async route handlers** must wrap in `catchAsync()` middleware. See: `src/app.ts:global error handler` and `src/utils/catchAsync.ts`.

### 2. Middleware Stack (in order)
1. `helmet()`: Security headers
2. `cors()`: Cross-origin config
3. `express.json()`: Body parser
4. **Routes** (decorated with auth, RBAC)
5. `jwtMiddleware()`: Token verification (if route protected)
6. `rbacMiddleware()`: Role-based access (ADMIN vs STAFF)
7. Global error handler

See: `src/app.ts:Express configuration`

### 3. Authentication & Authorization
- **JWT**: Tokens stored client-side, verified per request
- **Roles**: `ADMIN`, `STAFF` in User model
- **RBAC Middleware**: `src/middleware/rbac.middleware.ts` checks `req.user.role`

Protected route example: `inventory.routes.ts` → `router.post("/", rbacMiddleware("ADMIN"), ...)`

### 4. TypeScript Patterns
- Strict mode enabled; leverage types for safety
- **Controllers return**: `{ status, data, message }`
- **Services return**: Typed models (Prisma types auto-generated)
- **No `any`**: Use discriminated unions for varying response types

### 5. Real-time Events (Socket.IO)
Socket events broadcast inventory/sales changes:
```typescript
// src/socket.ts
io.emit("inventoryUpdated", { batchId, quantityRemaining });
io.emit("saleProcessed", { saleId, totalPrice, itemsCount });
```
**Frontend listens** in `SocketContext.tsx` → Updates UI instantly without polling.

---

## Critical Developer Workflows

### Database Setup
```bash
npm install                  # Install deps
npx prisma migrate dev      # Run migrations (auto-create `dist/`)
npm run seed                # Populate test data (if seed.ts exists)
```

### Local Development
**Backend** (`npm run dev`):
- Runs `nodemon` with `ts-node` transpilation
- Watches `src/` for changes; recompiles
- Listens on `http://localhost:5000`

**Frontend** (`client/ npm run dev`):
- Vite dev server with HMR
- Listens on `http://localhost:5173`
- Proxies API calls to `http://localhost:5000` (via vite.config.ts)

**Python AI Service** (manual start):
```bash
cd python_ai && pip install -r requirements.txt
python app.py  # Listens on http://localhost:8000
```

### Building for Production
**Backend**: `npm run build` → TypeScript → `dist/` → `npm start`
**Frontend**: `npm run build` → Vite → `client/dist/` (static files)

### PayPal Webhook Integration
- `paypal.controller.ts` receives `POST /api/v1/paypal/webhook`
- Verifies signature via `paypal.service.ts`
- Creates `Subscription` + `Payment` records
- No manual confirmation needed; webhook-driven

---

## Integration Points & External Services

### PayPal API (`src/services/paypal.service.ts`)
- **Environment**: `PAYPAL_CLIENT_ID`, `PAYPAL_CLIENT_SECRET`, `PAYPAL_MODE` (sandbox/live)
- **Endpoints used**: Subscription creation, payment verification
- **Error handling**: PayPal errors wrapped in `AppError`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sanjayr08/zenith-project](https://github.com/sanjayr08/zenith-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
