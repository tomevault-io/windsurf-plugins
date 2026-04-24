---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a headless e-commerce application with a Next.js frontend and Odoo 18 backend API module. The frontend communicates with Odoo via custom REST API endpoints for authentication, product catalog, cart management, and orders.

## Architecture

- **Frontend**: Next.js 16 with App Router (`ecommerce_frontend/`)
- **Backend**: Odoo 18 module providing REST API (`ecommerce_api/`)
- **State Management**: Zustand for client-side state
- **Styling**: Tailwind CSS with shadcn/ui components
- **API Communication**: Custom Odoo API client in `src/lib/api.ts`

## Common Commands

### Frontend (ecommerce_frontend/)
```bash
cd ecommerce_frontend
npm run dev          # Development server
npm run build        # Production build
npm start            # Production server
npm run lint         # ESLint
```

### Backend (ecommerce_api/)
This is an Odoo module. Development requires an Odoo 18 installation:
- Install the module in Odoo via Apps menu
- Module depends on: `base`, `website_sale`, `auth_signup`, `portal`, `sale`, `product`
- CORS configuration required for frontend access

## Key Files and Directories

### Frontend Structure
- `src/app/` - Next.js App Router pages and layouts
- `src/components/` - React components (layout, product, ui)
- `src/lib/api.ts` - Odoo API client with authentication, CRUD methods, and cart token management
- `src/lib/store.ts` - Zustand state management stores with cart persistence
- `src/lib/types.ts` - TypeScript type definitions
- `src/components/providers.tsx` - App initialization (session, cart token restoration)
- `.env.local` - Environment variables (copy from `.env.example`)

### Backend Structure
- `controllers/` - API endpoint implementations (auth, products, cart, orders, user)
- `controllers/main.py` - Core utilities: CORS, rate limiting, security headers, validation
- `models/` - Odoo model extensions and API configuration
- `models/cart_token.py` - Cart token model for guest cart persistence
- `security/` - Access control definitions
- `data/api_config_data.xml` - Default config and cron jobs
- `__manifest__.py` - Module configuration with API endpoint documentation

## API Endpoints

The backend provides RESTful endpoints under `/api/v1/`:

**Authentication:**
- `POST /api/v1/auth/login` - Login with optional cart_token for cart merge
- `POST /api/v1/auth/register` - Register with optional cart_token, auto-links guest orders
- `POST /api/v1/auth/logout`
- `GET /api/v1/auth/session`
- `POST /api/v1/auth/reset-password`

**Products:**
- `GET /api/v1/products`
- `GET /api/v1/products/<id>`
- `GET /api/v1/products/featured`
- `GET /api/v1/products/search-suggestions`
- `GET /api/v1/categories`

**Cart:**
- `GET /api/v1/cart`
- `POST /api/v1/cart/add`
- `POST /api/v1/cart/update`
- `DELETE /api/v1/cart/remove/<line_id>`
- `DELETE /api/v1/cart/clear`
- `GET /api/v1/cart/count`
- `GET /api/v1/cart/token` - Get/create cart token for persistence
- `POST /api/v1/cart/restore` - Restore cart from token
- `POST /api/v1/cart/merge` - Merge guest cart into user's cart

**Orders:**
- `GET /api/v1/orders`
- `GET /api/v1/orders/<id>`
- `POST /api/v1/orders/create`
- `POST /api/v1/orders/<id>/cancel`
- `POST /api/v1/orders/<id>/reorder`
- `POST /api/v1/orders/<id>/restore`

**User Profile:**
- `GET /api/v1/user/profile`
- `POST /api/v1/user/profile`
- `GET /api/v1/user/addresses`
- `POST /api/v1/user/addresses`
- `PUT /api/v1/user/addresses/<id>`
- `DELETE /api/v1/user/addresses/<id>`
- `POST /api/v1/user/change-password`

**Utilities:**
- `GET /api/v1/countries`
- `GET /api/v1/health`

## Security Features

The API implements comprehensive security measures:

### Rate Limiting
- Login: 5 attempts per minute (with progressive lockout)
- Registration: 3 attempts per minute
- Password reset: 3 attempts per 5 minutes
- Password change: 3 attempts per minute

### Input Validation
- Email format validation (RFC 5322)
- Password strength requirements (8+ chars, uppercase, lowercase, number)
- Quantity bounds checking (0.01 - 9999)
- String sanitization (HTML/script tag removal)

### Session Security
- HttpOnly cookies with SameSite=Lax
- 24-hour session expiry
- Cart ownership verification on all operations

### Security Headers
All API responses include:
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY`
- `X-XSS-Protection: 1; mode=block`
- `Referrer-Policy: strict-origin-when-cross-origin`
- `Cache-Control: no-store` (for sensitive endpoints)

### CORS
- Configured allowed origins in `ecommerce.api.config`
- No wildcard fallback (secure by default)

## Guest Cart Persistence

The system supports persistent guest carts that survive session expiry:

### How It Works
1. **Token Generation**: When a guest adds items, a UUID token is created and stored
2. **Storage**: Token saved in localStorage + cookie (30-day expiry)
3. **Restoration**: On return visit, cart is restored via token
4. **Transfer**: On login/register, guest cart is merged/transferred to user's account
5. **Order Linking**: Past guest orders are auto-linked when user registers with same email


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FadhelAlqaidoomoi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
