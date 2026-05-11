---
trigger: always_on
description: SubTrackr is a self-hosted subscription management application built with Go and HTMX. It helps users track subscriptions, visualize spending, and get renewal reminders.
---

# SubTrackr - Agent Documentation

## Project Overview

SubTrackr is a self-hosted subscription management application built with Go and HTMX. It helps users track subscriptions, visualize spending, and get renewal reminders.

## Architecture

### Tech Stack
- **Backend**: Go 1.21+ with Gin web framework
- **Database**: SQLite (GORM)
- **Frontend**: HTMX + Tailwind CSS
- **Deployment**: Docker & Docker Compose

### Project Structure

```
subtrackr-xyz/
├── cmd/
│   ├── server/          # Main server entry point
│   └── migrate-dates/   # Date migration utility
├── internal/
│   ├── config/          # Configuration management
│   ├── database/        # Database initialization and migrations
│   ├── handlers/        # HTTP request handlers (Gin handlers)
│   ├── middleware/      # HTTP middleware (auth, etc.)
│   ├── models/          # Data models (GORM models)
│   ├── repository/      # Data access layer
│   ├── service/         # Business logic layer
│   └── version/         # Version information
├── templates/           # HTML templates (HTMX)
├── web/static/          # Static assets (JS, CSS, images)
├── tests/               # Playwright E2E tests
└── data/                # SQLite database (gitignored)
```

### Key Components

#### 1. Server Entry Point (`cmd/server/main.go`)
- Initializes database, repositories, services, and handlers
- Sets up Gin router with templates
- Configures routes (web and API)
- Starts HTTP server

#### 2. Handlers (`internal/handlers/`)
- **subscription.go**: CRUD operations for subscriptions
- **settings.go**: SMTP config, Pushover config, notifications, API keys, currency, dark mode
- **category.go**: Category management

#### 3. Services (`internal/service/`)
- Business logic layer
- **subscription.go**: Subscription operations
- **settings.go**: Settings management
- **category.go**: Category operations
- **currency.go**: Currency conversion (Fixer.io integration)
- **email.go**: Email notification service (SMTP)
- **pushover.go**: Pushover notification service

#### 4. Models (`internal/models/`)
- GORM models:
  - `Subscription`: Main subscription entity
  - `Category`: Subscription categories
  - `Settings`: Application settings (key-value store)
  - `SMTPConfig`: Email configuration
  - `PushoverConfig`: Pushover notification configuration
  - `APIKey`: API authentication keys
  - `ExchangeRate`: Currency exchange rates

#### 5. Repository (`internal/repository/`)
- Data access layer using GORM
- Abstracts database operations

### Routing Structure

#### Web Routes (HTMX)
- `/` - Dashboard
- `/dashboard` - Dashboard
- `/subscriptions` - Subscription list
- `/analytics` - Analytics view
- `/settings` - Settings page
- `/form/subscription` - Subscription form modal

#### API Routes (HTMX)
- `/api/subscriptions` - Subscription CRUD
- `/api/stats` - Statistics
- `/api/export/*` - Data export
- `/api/settings/*` - Settings management
- `/api/categories` - Category management

#### Public API Routes (Require API Key)
- `/api/v1/subscriptions` - Subscription CRUD
- `/api/v1/stats` - Statistics
- `/api/v1/export/*` - Data export

### Database Schema

#### Subscriptions
- ID, Name, Cost, OriginalCurrency
- Schedule: Monthly, Annual, Weekly, Daily
- Status: Active, Cancelled, Paused, Trial
- CategoryID (foreign key)
- Dates: StartDate, RenewalDate, CancellationDate
- Additional: PaymentMethod, Account, URL, Notes, Usage

#### Categories
- ID, Name
- CreatedAt, UpdatedAt

#### Settings
- Key-value store for application settings
- Keys: `smtp_config`, `renewal_reminders`, `currency`, etc.

### Key Features

1. **Subscription Management**
   - CRUD operations
   - Multiple schedules (Monthly, Annual, Weekly, Daily)
   - Categories
   - Multi-currency support

2. **Email Notifications**
   - SMTP configuration with TLS/SSL support
   - STARTTLS for ports 2525, 8025, 587, 25, 80
   - Implicit TLS for ports 465, 8465, 443
   - Renewal reminders
   - High cost alerts

3. **Pushover Notifications**
   - Pushover API integration for mobile push notifications
   - User Key and Application Token configuration
   - Renewal reminders (same settings as email)
   - High cost alerts (same threshold as email)
   - Works alongside email notifications

4. **Currency Support**
   - USD, EUR, GBP, JPY, RUB, SEK, PLN, INR, CHF, BRL, COP, BDT
   - Optional Fixer.io integration for real-time rates
   - Automatic conversion display
   - BDT (Bangladeshi Taka) with ৳ symbol

5. **API Access**
   - API key authentication
   - RESTful endpoints
   - JSON responses

5. **Data Management**
   - CSV/JSON export
   - Backup functionality
   - Clear all data option

### Development Guidelines

#### Code Style
- Follow Go standard formatting (`go fmt`)
- Use meaningful variable and function names
- Add comments for exported functions
- Keep functions focused and small

#### Error Handling
- Return errors from functions, don't panic
- Log errors appropriately
- Provide user-friendly error messages in handlers

#### Testing
- Unit tests in `*_test.go` files
- E2E tests in `tests/` using Playwright
- Test API endpoints with `test-api.sh`

#### Database Migrations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bscott/subtrackr](https://github.com/bscott/subtrackr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
