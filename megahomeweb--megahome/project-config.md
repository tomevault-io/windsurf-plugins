---
trigger: always_on
description: Uzbek wholesale e-commerce platform ("MegaHome ulgurji" = MegaHome wholesale). Built with Next.js 16 App Router, Firebase backend, Zustand state management, Tailwind CSS 4. Features ERP-like capabilities: stock management, credit tracking (nasiya), invoicing, profit reports, Excel import/export, and Remotion video marketing.
---

# MegaHome Ulgurji - E-Commerce Platform

## Project Overview
Uzbek wholesale e-commerce platform ("MegaHome ulgurji" = MegaHome wholesale). Built with Next.js 16 App Router, Firebase backend, Zustand state management, Tailwind CSS 4. Features ERP-like capabilities: stock management, credit tracking (nasiya), invoicing, profit reports, Excel import/export, and Remotion video marketing.

## Tech Stack
- **Framework**: Next.js 16.1.1 (App Router, React 19)
- **Language**: TypeScript 5
- **Styling**: Tailwind CSS 4 + Radix UI + Headless UI + shadcn/ui (new-york style)
- **State**: Zustand 5.0.4 (10 persisted/reactive stores)
- **Backend**: Firebase 11.8.1 (Auth + Firestore + Storage)
- **Server**: Firebase Admin 13.4.0 (API routes)
- **Forms**: React Hook Form + Zod validation
- **Email**: Nodemailer (Gmail SMTP)
- **Video**: Remotion (marketing video production)
- **Icons**: Lucide React + React Icons
- **Animations**: AOS (Animate On Scroll)
- **Notifications**: React Hot Toast
- **Data**: xlsx (Excel/CSV import-export), uuid

## Directory Structure
```
app/
├── (auth)/           # Login & signup pages
├── (client-side)/    # Customer-facing pages (home, product, category, cart, orders)
├── admin/            # Admin panel (dashboard, products, categories, orders, customers,
│                     #   nasiya, kirim, invoices, reports, profile)
├── api/              # API routes (delete-user, sendOrderEmail)
├── layout.tsx        # Root layout (AuthProvider, Toaster, fonts)
└── globals.css       # Tailwind theme + custom CSS variables
components/
├── admin/            # Admin UI (Sidebar, Tables, Forms, Search, BulkPriceUpdate,
│                     #   ImportProducts, DashboardSummary, DailySummary, Notifications)
├── auth/             # LoginForm, SignUpForm, ProtectedRoute
├── client/           # Customer UI (Header, Footer, ProductCard, Cart, Modal, Quantity)
├── contents/         # Page content wrappers
└── ui/               # shadcn/ui primitives (button, avatar, dialog, etc.)
store/                # Zustand stores (10 stores)
├── authStore.ts      # Auth state, user data, role checks
├── useCartStore.ts   # Cart items, quantities, totals (persisted)
├── useCategoryStore.ts # Category CRUD with real-time listeners
├── useProductStore.ts  # Product CRUD with real-time listeners
├── useOrderStore.ts  # Order management with status tracking
├── useDraftStore.ts  # Auto-save drafts (24h expiry)
├── useNasiyaStore.ts # Credit/debt tracking
├── useNotificationStore.ts # Admin notifications
├── useStockReceiptStore.ts # Stock receipt (kirim) management
└── useToggleStore.ts # Sidebar toggle state
remotion/             # Video marketing production
├── Root.tsx          # Composition registry
├── styles.ts         # Brand colors, typography, animation presets
├── index.ts          # Remotion Studio entry point
└── compositions/     # Video templates
    ├── ProductShowcase.tsx   # Product image carousel video (5s)
    ├── PromoVideo.tsx        # Sale/discount promo video (8s)
    └── CategoryHighlight.tsx # Category showcase video (6s)
firebase/config.ts    # Firebase client SDK initialization
lib/
├── types.ts          # TypeScript interfaces (ProductT, CategoryI, Order, UserData, ImageT)
├── utils.ts          # Utility functions (cn() class merging)
├── formatPrice.ts    # Price formatting (UZS)
├── exportExcel.ts    # Excel export utilities
├── importExcel.ts    # Excel import with smart column detection (UZ/EN/RU)
├── orderStatus.ts    # Order status constants
└── notificationSound.ts # Web Audio API notifications
hooks/                # Custom hooks (useWhiteBody, useScrollBodyColor)
providers/            # AuthProvider (Firebase onAuthStateChanged)
```

## Firebase Collections
- **"user"**: `{name, email, uid, role: "admin"|"user", phone, time, date}`
- **"categories"**: `{id, name, description, categoryImgUrl[], storageFileId, subcategory[]}`
- **"products"**: `{id, title, price, costPrice?, productImageUrl[], category, description, quantity, stock?, time, date, storageFileId, subcategory}`
- **"orders"**: `{id, clientName, clientPhone, date, basketItems[], totalPrice, totalQuantity, userUid, status?}`
- **"nasiya"**: Credit/debt tracking records
- **"receipts"**: Stock receipt documents
- **"invoices"**: Generated invoices (schyot-faktura)

## Auth Flow
1. Signup creates Firebase Auth user + Firestore "user" doc (default role: "user")
2. Login authenticates via Firebase → fetches user data → stores in Zustand
3. ProtectedRoute checks `isAuthenticated` and `isAdmin()` for admin routes
4. AuthProvider uses `onAuthStateChanged` listener for session persistence

## Key Patterns
- Real-time Firestore listeners (`onSnapshot`) for products/categories
- Zustand stores with `persist` middleware for cart/auth
- Product images stored in Firebase Storage at `products/{storageFileId}/`
- Category images at `categories/{storageFileId}/`
- Draft auto-save with 24-hour expiry and storage cleanup
- Role-based routing: admin → /admin, user → /

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/megahomeweb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
