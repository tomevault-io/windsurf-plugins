---
trigger: always_on
description: **Balance Cloud App** is a Laravel 12 + Vue.js 3 financial web application for personal checkbook management. Users can track accounts, transactions, and financial data through both web interface and admin dashboard. The application uses Laravel Passport for authentication, PrimeVue for UI components, and modern tooling including Vite and TailwindCSS.
---

# Copilot Instructions for Balance Cloud App

## Project Overview
**Balance Cloud App** is a Laravel 12 + Vue.js 3 financial web application for personal checkbook management. Users can track accounts, transactions, and financial data through both web interface and admin dashboard. The application uses Laravel Passport for authentication, PrimeVue for UI components, and modern tooling including Vite and TailwindCSS.

**Repository Size**: ~50MB (excluding node_modules/vendor)  
**Tech Stack**: PHP 8.2+, Laravel 12, Vue.js 3, MySQL, Node.js, Vite, TailwindCSS 4, PrimeVue 4  
**Primary Purpose**: Personal finance management with multi-user support and admin interface

## Build & Development Commands

### Environment Setup (CRITICAL - Do This First)
```bash
# Always install dependencies before any build/test operations
composer install
npm install

# Copy environment file (required for all operations)
cp .env.example .env

# Configure .env with database credentials and generate app key
php artisan key:generate

# Setup database (MySQL required)
php artisan migrate
php artisan passport:install
```

### Build Commands (Validated Working)
```bash
# Development build with hot reload
npm run dev                    # Starts Vite dev server
php artisan serve             # Starts PHP server on :8000

# Production build (always works)
npm run build                 # Takes ~4-5 seconds, generates optimized assets
npm run prod                  # Alias for build

# Combined development server
npm run serve                 # Runs both PHP server and Vite dev concurrently
```

### Testing (Known Issues & Workarounds)
```bash
# WORKING: Run Laravel tests
php artisan test              # Standard Laravel test runner
php artisan test --stop-on-failure

# ISSUE: Some tests fail with 418 status (SSL requirement)
# WORKAROUND: Tests require proper .env.testing configuration
# Ensure APP_URL=http://localhost:8000 in .env.testing

# Test database setup required:
php artisan migrate:fresh --env=testing
```

### Build Validation Steps
1. **Always run `composer install && npm install` first**
2. **Build assets**: `npm run build` (must complete without errors)
3. **Test server**: `php artisan serve` (should start on :8000)  
4. **Clear caches**: `php artisan optimize:clear` (if encountering issues)

### Common Build Errors & Solutions
- **"Mix manifest not found"**: Run `npm run build` first
- **Tests failing with 418**: Check .env.testing SSL configuration  
- **Composer deprecation warnings**: Safe to ignore (PHP 8.4 compatibility issue)
- **Vue/Vite errors**: Ensure Node.js 16+ and clean `npm install`

## Architecture & Project Layout

### Directory Structure
```
app/
├── Http/Controllers/Api/Admin/     # Admin API controllers
├── Http/Controllers/Api/           # User API controllers  
├── Models/                         # Eloquent models
├── Helpers/                        # Custom helpers (WithHelper, SearchHelper, PaginationHelper)
└── Services/                       # Business logic services

resources/
├── js/admin/                       # Vue.js admin interface
│   ├── views/                      # Vue components
│   ├── router.js                   # Vue router config
│   └── api.js                      # API client
├── css/                           # Stylesheets
└── views/                         # Blade templates

routes/
├── api.php                        # User API routes  
├── admin_api.php                  # Admin API routes
└── web.php                        # Web routes

config/                            # Laravel configuration
tests/Feature/                     # Feature tests
tests/Unit/                        # Unit tests
```

### Key Configuration Files
- **vite.config.js**: Asset bundling, includes image copying
- **.eslintrc.js**: Vue.js code standards, Airbnb base + Vue 3 rules  
- **tailwind.config.js**: CSS framework configuration
- **phpunit.xml**: Test configuration
- **composer.json**: PHP dependencies & scripts
- **package.json**: Node.js dependencies & build scripts

### Database Models & Relationships
**Core Models**: User, Account, Transaction, TransactionName, AccountType, CurrentBalance  
**Admin Models**: ContactUs, WhatsNew, UserProfile  
**Auth**: Laravel Passport OAuth clients & tokens

**Key Relationships**:
- Users have many Accounts, Transactions, TransactionNames
- Accounts belong to Users and AccountTypes  
- Transactions belong to Users and reference TransactionNames
- CurrentBalance tracks real-time account totals

### API Architecture
**User API** (`/api/`): User-owned resource management  
**Admin API** (`/api/admin/v1/`): Cross-user administration, analytics, dashboard

**Authentication**: Laravel Passport with both Bearer tokens and cookie-based auth  
**Middleware**: `auth:api`, `scope:dashboard`, `role:admin`, `log_ip`

### Frontend Architecture  
**Vue.js 3 SPA** with Composition API  
**PrimeVue 4 Components**: DataTable, Form, Card, Toast, etc.  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheBootstrapParadox/balancecloud](https://github.com/TheBootstrapParadox/balancecloud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
