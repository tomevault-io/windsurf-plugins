---
trigger: always_on
description: > Mobile app for ordering construction vehicles, equipment, and readymix materials.
---

# RMC Readymix — CLAUDE.md

> Mobile app for ordering construction vehicles, equipment, and readymix materials.
> Full BRD: `DOCS/BRD/RMC_Readymix_BRD.docx`

---

## Quick Start

```bash
# Terminal 1 — Backend (MUST use Bun, not Node)
cd backend
bun run src/seed/seed.js     # first-time only: wipes + re-seeds DB
bun --watch src/index.js     # dev server with hot reload → http://localhost:3000

# Terminal 2 — Mobile app
cd mobile
npx expo start --ios         # iOS Simulator (localhost works natively)
```

**Test credentials**
| Role | Email | Password |
|------|-------|----------|
| Admin | `admin@rmc.com` | `admin123` |
| User | `user@rmc.com` | `user123` |
| User 2 | `rahul@builders.com` | `user123` |

---

## ⚠️ Critical: Runtime Constraint

**Node v24.11.1 is installed.** All native SQLite npm packages (`better-sqlite3`, `sqlite3`, etc.) fail to compile on Node v24 — no prebuilt binaries exist yet for this version.

**Fix:** The backend runs with **Bun v1.3.9**, which ships built-in `bun:sqlite` — a synchronous, file-persistent SQLite API nearly identical to `better-sqlite3`. All backend scripts must use `bun run` or `bun --watch`, never `node` or `nodemon`.

---

## Project Structure

```
rmc-mobile-app/
├── CLAUDE.md                              ← you are here
├── .gitignore                             # ignores node_modules, *.db, .expo/
├── DOCS/BRD/
│   └── RMC_Readymix_BRD.docx             # Full business requirements document
│
├── backend/                              # Express REST API — run with Bun
│   ├── package.json                      # all scripts use "bun run …"
│   ├── rmc.db                            # SQLite database file (gitignored)
│   └── src/
│       ├── index.js                      # Entry: mounts all routes, port 3000
│       ├── config/
│       │   ├── database.js               # bun:sqlite connection + initDB() (CREATE TABLE IF NOT EXISTS)
│       │   └── auth.js                   # JWT_SECRET, TOKEN_EXPIRY = '7d'
│       ├── middleware/
│       │   ├── auth.js                   # Verifies Bearer token → attaches req.user
│       │   ├── adminOnly.js              # Blocks non-admin roles with 403
│       │   └── errorHandler.js           # Global Express error handler
│       ├── models/                       # Raw SQL via db.prepare() — synchronous
│       │   ├── user.model.js             # findByEmail, findById, create, emailExists
│       │   ├── product.model.js          # findAll (filtered), findById, findFeatured,
│       │   │                             # getCategories, findAllAdmin, findByIdAdmin,
│       │   │                             # create, update, toggleActive
│       │   ├── cart.model.js             # getByUserId, findItem, addItem (upsert),
│       │   │                             # updateQuantity, removeItem, clearCart
│       │   └── order.model.js            # create, findByUserId, findById, findByIdAndUser,
│       │                                 # findAll (with customer JOIN), updateStatus,
│       │                                 # getNextInvoiceSequence
│       ├── controllers/
│       │   ├── auth.controller.js        # register, login, getProfile
│       │   ├── product.controller.js     # getProducts, getProductById, getCategories, getFeatured
│       │   ├── cart.controller.js        # getCart, addToCart, updateCartItem,
│       │   │                             # removeCartItem, clearCart
│       │   ├── order.controller.js       # checkout, getOrders, getOrderById
│       │   └── admin.controller.js       # getAllOrders, getOrderDetail, updateOrderStatus,
│       │                                 # getAllProducts, getProductDetail, createProduct,
│       │                                 # updateProduct, toggleProductActive
│       ├── routes/
│       │   ├── auth.routes.js            # /api/v1/auth/*
│       │   ├── product.routes.js         # /api/v1/products/*   (public)
│       │   ├── cart.routes.js            # /api/v1/cart/*       (user JWT required)
│       │   ├── order.routes.js           # /api/v1/orders/*     (user JWT required)
│       │   └── admin.routes.js           # /api/v1/admin/*      (admin JWT required)
│       ├── utils/
│       │   ├── invoice.js                # generateInvoiceNumber() → "RMC/YYYY-MM/XXXXX"
│       │   └── validators.js             # isValidEmail, isValidPassword, isValidPhone
│       └── seed/
│           └── seed.js                   # Wipes all data; inserts 3 users, 6 categories, 13 products
│
└── mobile/                               # React Native + Expo Router
    ├── package.json                      # main: "expo-router/entry"
    ├── app.json                          # name: "RMC Readymix", slug: "rmc-readymix", scheme: "rmc"
    ├── tsconfig.json                     # strict: true, paths: { "@/*": ["./*"] }
    ├── app/                              # File-based routes (Expo Router)
    │   ├── _layout.tsx                   # Root: wraps AuthProvider + CartProvider,
    │   │                                 # auth redirect via useSegments + useRouter
    │   ├── (auth)/
    │   │   ├── _layout.tsx               # Stack with saffron header
    │   │   ├── login.tsx                 # Email/password login + test credentials hint

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lalit2001) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
