---
trigger: always_on
description: This file is the **single source of truth** for AI agents contributing to this project.
---

# NOVA Store — Agent Guide

This file is the **single source of truth** for AI agents contributing to this project.
Read it before making any changes. If anything is unclear, search the codebase before inventing conventions.

---

## 1. Project Overview

**NOVA Store** is a didactic fullstack e-commerce monorepo: a static HTML/CSS/JS frontend and an ASP.NET Core 8 Web API backed by SQL Server.

```
Browser (static HTML, port 5500)
       |
   fetch()  ←───  REST API (http://localhost:5036)
       |
  ASP.NET Core 8
       |
  Entity Framework Core + ASP.NET Identity
       |
  SQL Server (NovaStoreDb)
```

| Layer | Technology | Notes |
|-------|-----------|-------|
| Frontend | HTML5, CSS3, Vanilla JS | No build step. Multi-page app (not SPA) |
| Frontend libs | Bootstrap 5.3.3 CSS, Bootstrap Icons 1.11.3, Google Fonts "Inter" | Bootstrap grid/utilities; Bootstrap JS only where pages already load it |
| Backend | ASP.NET Core 8, C# 12 | `.NET 8.0` SDK (`global.json`) |
| ORM | Entity Framework Core 8 | Code-first, Fluent API, manual DTO mapping (no AutoMapper) |
| Auth | ASP.NET Identity + JWT Bearer | Roles: `Admin`. OAuth: Google + Microsoft |
| Payments | Stripe Checkout | `PaymentsController`, webhook optional locally |
| Email | SMTP via `EmailService` | Mailtrap in dev; empty `SmtpHost` logs reset link to console |
| Database | SQL Server | Connection string in `appsettings.json` |
| API docs | Swagger / Swashbuckle | `/swagger` in Development |

### Implemented features (work packages)

| WP | Feature | Status |
|----|---------|--------|
| Core | Products, categories, cart, checkout, orders | Done |
| Auth | Register, login, profile, JWT | Done |
| OAuth | Google login, Microsoft login | Done |
| Payments | Stripe Checkout + order verify | Done |
| WP1 | Forgot/reset password email | Done |
| WP2 | Product search (`?search=`, header search) | Done |
| WP3 | Admin panel (roles, CRUD, dashboard) | Done |
| WP4 | Transactional emails (order confirmation, welcome) | Done |
| Post-WP4 | Auth-required checkout + order history (`orders.html`) | Done |

### Team workflow

- **OpenCode (DeepSeek)** — plans features and writes execution specs
- **Cursor** — implements code following this guide
- Work package order: **WP1 → WP2 → WP3 → WP4**

---

## 2. Dev Environment

### Quick start (Windows)

```bat
start-dev.bat
```

Opens two CMD windows:
- Backend: `http://localhost:5036` (Swagger at `/swagger`)
- Frontend: `http://localhost:5500`

Scripts live in `scripts/` (`start-backend.bat`, `start-frontend.bat`). Use **CMD**, not PowerShell, when paths contain spaces (`Ecommerce project - 1`).

### Manual start

```bash
# Backend
cd backend/NovaStore.Api
dotnet ef database update    # first time / after new migration
dotnet run --launch-profile http

# Frontend
cd frontend
python -m http.server 5500
```

### URLs & config

| What | Value |
|------|-------|
| API base (frontend) | `http://localhost:5036/api` — defined in `frontend/assets/js/api.js` as `API_BASE` |
| Frontend | `http://localhost:5500` |
| CORS policy name | `FrontendCors` — allows all origins in Development |
| Admin seed user | `admin@novastore.hr` / `Admin123!` (from `Admin` section in appsettings) |

**Secrets:** `appsettings.json` contains dev keys (Stripe, Mailtrap, JWT). Do not commit production secrets. Do not paste real secrets into chat or docs.

---

## 3. Project Structure

```
Ecommerce project - 1/
├── AGENTS.md                  # ← you are here
├── README.md
├── start-dev.bat
├── start-dev.ps1
├── scripts/
│   ├── start-backend.bat
│   └── start-frontend.bat
│
├── frontend/                  # STATIC CLIENT (no build)
│   ├── index.html
│   ├── shop.html              # filters, sort, ?search=
│   ├── product.html
│   ├── cart.html              # localStorage
│   ├── checkout.html          # Stripe checkout
│   ├── order-success.html
│   ├── categories.html
│   ├── about.html
│   ├── contact.html
│   ├── login.html             # email + Google + Microsoft
│   ├── register.html
│   ├── profile.html
│   ├── orders.html            # order history (requires auth)
│   ├── forgot-password.html
│   ├── reset-password.html
│   ├── admin.html             # dashboard
│   ├── admin-products.html
│   ├── admin-orders.html
│   ├── admin-categories.html
│   ├── admin-users.html
│   └── assets/
│       ├── css/style.css      # single stylesheet (design system + admin)
│       └── js/
│           ├── products.js    # mock data + getters (fallback)
│           ├── api.js         # API_BASE, fetchProducts, fetchMyOrders(), loadStoreData()
│           ├── auth.js        # JWT, authFetch, isAdmin(), requireAuth(), getLoginRedirectUrl()
│           ├── cart.js        # localStorage cart, formatPrice()
│           ├── main.js        # header/footer, NAV_LINKS, productCardHTML
│           ├── admin.js       # requireAdmin(), adminFetch(), admin layout
│           └── msal-browser.min.js   # Microsoft login (local, not CDN)
│
└── backend/
    ├── global.json
    ├── NovaStore.sln
    └── NovaStore.Api/
        ├── Program.cs         # DI, CORS, JWT, Identity, Stripe, AdminSeeder
        ├── appsettings.json
        ├── appsettings.Development.json
        ├── Controllers/
        │   ├── ProductsController.cs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jure135/EcommerceSite-1](https://github.com/Jure135/EcommerceSite-1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
