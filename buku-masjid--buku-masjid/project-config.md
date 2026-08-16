---
trigger: always_on
description: This document contains essential information for understanding the architecture, setup, commands, and coding patterns of the Buku Masjid application.
---

# Buku Masjid - Development Guide

This document contains essential information for understanding the architecture, setup, commands, and coding patterns of the Buku Masjid application.

---

## 1. Project Overview

[Buku Masjid](https://github.com/buku-masjid/buku-masjid) is a web-based financial management and lecturing schedule system for mosques (masjid/mushalla) built with Laravel. It serves both authenticated mosque administrators and unauthenticated public visitors.

**Core objectives:**
- **Transparency**: Publish cash/financial reports online for congregation and public.
- **Convenience**: Simplify income/spending transaction logging for treasurers.
- **Automation**: Auto-generate monthly, weekly, and categorized financial reports.
- **Scheduling**: Manage routine lecturing (pengajian) and Friday khatib schedules.

---

## 2. Tech Stack

| Layer | Technology |
|---|---|
| Backend | PHP `^8.1`, Laravel `10.x` |
| Database | MySQL / MariaDB (production), SQLite in-memory (tests) |
| Frontend | Bootstrap `4.0.0`, SCSS/Sass, Vanilla JS |
| Asset Bundler | Laravel Mix `6.x` (Webpack) |
| Reactive UI | Livewire `2.x` |
| API Auth | Laravel Passport `11.x` (OAuth2) |
| Testing | PHPUnit `10.x`, Laravel BrowserKit Testing |
| Code Style | Laravel Pint (Laravel preset) |
| Deployment | Deployer (`deploy.php`) |
| Package Manager | Yarn (see `.nvmrc` for Node version) |

---

## 3. Development Commands

### Initial Setup

```bash
cp .env.example .env
php artisan key:generate
php artisan passport:keys          # Required for API authentication
php artisan storage:link           # Link public disk for file uploads
php artisan migrate --seed         # Creates tables + seeds default user/book/categories
```

**Default credentials after seeding:**
```
URL:      http://localhost:8000
Email:    admin@example.net
Password: password
```

### Local Server

```bash
php artisan serve
```

### Database

```bash
php artisan migrate --seed                           # Fresh migration with seed
php artisan buku-masjid:generate-demo-data           # Generate dummy data (last 3 months)
php artisan buku-masjid:remove-demo-data             # Remove dummy data (where created_at IS NULL)
php artisan partner:generate {type_code} {--count=} {--reset}  # Generate fake partner records
php artisan partner:upgrade-type-levels              # One-time migration for partner type/level format
```
> **Note:** Custom commands can be found in `app/Console/Commands` or as closures in `routes/console.php`.

### Frontend Assets

```bash
yarn                   # Install JS dependencies (use yarn, not npm)
npm run dev            # Development build (one-time)
npm run watch          # Watch mode for development
npm run prod           # Production build
```

### Testing

```bash
vendor/bin/phpunit                    # Run all tests (395 tests, uses SQLite in-memory)
vendor/bin/phpunit --filter TestName  # Run specific test
```

> **Note:** Tests run on SQLite in-memory (configured in `phpunit.xml`). No database setup needed for tests.

### Code Quality

```bash
vendor/bin/pint          # Fix code style issues
vendor/bin/pint --test   # Check code style without fixing (used in CI)
```

---

## 4. Key Environment Variables (`.env`)

All variables below come from `.env.example`. Critical ones to configure:

### Application
```dotenv
APP_NAME="Buku Masjid"
APP_ENV=local
APP_URL=http://localhost
APP_TIMEZONE="Asia/Makassar"    # Affects Carbon date/time calculations
```

### Database
```dotenv
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret
```

### Mosque Identity
```dotenv
MASJID_NAME="Masjid Ar-Rahman"
MASJID_DEFAULT_BOOK_ID=1         # Fallback active book
AUTH_DEFAULT_PASSWORD=password   # Default password used by seeder
```

### Money Formatting
```dotenv
MONEY_CURRENCY_CODE="Rp"
MONEY_CURRENCY_TEXT="Rupiah"
MONEY_PRECISION=2
MONEY_DECIMAL_SEPARATOR=","
MONEY_THOUSANDS_SEPARATOR="."
```

### Partners (Congregation Members/Donors)
```dotenv
PARTNER_TYPES="donatur|Donatur"                            # Pipe-delimited: code|label
PARTNER_LEVELS="donatur:silver|Silver|gold|Gold|platinum|Platinum"           # typeCode:levelCode|label,...
PARTNER_INCOME_DEFAULT_VALUE="Hamba Allah"
PARTNER_SPENDING_DEFAULT_VALUE="Tanpa Nama"
```

### Feature Flags
```dotenv
FEATURES_LECTURINGS_IS_ACTIVE=true       # Enable lecturing/schedule module
FEATURES_DONORS_IS_ACTIVE=true           # Enable donor sub-module (subset of partners)
FEATURES_SHALAT_TIME_IS_ACTIVE=true      # Enable shalat time display on public page
FEATURES_PUBLIC_DISPLAY_IS_ACTIVE=false  # Enable /display TV screen page
```

### Shalat Time Integration
```dotenv
SHALAT_TIME_PROVIDER=myquran_api
MYQURAN_CITY_NAME="Kota Banjarmasin"    # Must match a city name in MyQuran API
```

### Public Display (TV Screen Mode)
```dotenv
PUBLIC_DISPLAY_THEME=default   # or "light"
```

### Disk & Queue
```dotenv
DISK_QUOTA=1GB             # Used by DiskUsageService to warn when storage is full
QUEUE_DRIVER=sync          # Change to "database" or "redis" for async image optimization
FILESYSTEM_DRIVER=public   # File uploads go to storage/app/public
```

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [buku-masjid/buku-masjid](https://github.com/buku-masjid/buku-masjid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
