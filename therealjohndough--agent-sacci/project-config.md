---
trigger: always_on
description: This file provides essential context for AI assistants working on this repository.
---

# CLAUDE.md — Sacci Brand Hub

This file provides essential context for AI assistants working on this repository.

## Project Overview

**Sacci Brand Hub** is a lightweight PHP 8.2 CMS (Content Management System) built for House of Sacci, a cannabis retailer. It provides:

- Internal brand portal for staff to manage tickets, assets, and content blocks
- Retail partner portal for organization-specific asset access
- Role-based access control (RBAC) for multi-tenant access management
- Digital asset management with controlled file downloads

**Deployment target:** SiteGround shared hosting (no Docker, no Node.js build step, no heavy framework).

---

## Repository Structure

```
agent-sacci/
├── sacci_brand_hub/          # Main application root (document root on server)
│   ├── index.php             # Front controller — registers all routes and boots app
│   ├── composer.json         # Dependency manifest (PHPMailer only)
│   ├── .htaccess             # Apache: routes all requests through index.php
│   ├── config/
│   │   └── config.php        # loadEnv() and env() helpers — reads .env file
│   ├── core/                 # Framework-like base classes
│   │   ├── Auth.php          # Static session auth + RBAC permission checks
│   │   ├── Csrf.php          # Per-session CSRF token generation & validation
│   │   ├── Database.php      # Singleton PDO wrapper (MySQL 8, utf8mb4)
│   │   ├── Router.php        # Simple method+path → controller dispatch
│   │   └── View.php          # Output-buffered PHP template renderer with layout
│   ├── app/
│   │   ├── controllers/      # One controller per endpoint group
│   │   │   ├── BaseController.php      # requireLogin(), render(), csrfToken()
│   │   │   ├── AuthController.php      # GET+POST /login, GET /logout
│   │   │   ├── DashboardController.php # GET /app
│   │   │   ├── TicketController.php    # GET /tickets
│   │   │   ├── AssetController.php     # GET /assets, GET /assets/download
│   │   │   ├── ContentController.php   # GET /content (admin-only)
│   │   │   └── PortalController.php    # GET /portal (retailer-only)
│   │   ├── models/           # Active-record style models extending BaseModel
│   │   │   ├── BaseModel.php           # find(), findBy(), create(), update()
│   │   │   ├── User.php
│   │   │   ├── Role.php
│   │   │   ├── Permission.php
│   │   │   ├── Ticket.php
│   │   │   ├── TicketComment.php
│   │   │   ├── TicketActivity.php
│   │   │   ├── Asset.php
│   │   │   ├── ContentBlock.php
│   │   │   ├── Organization.php
│   │   │   ├── Setting.php
│   │   │   └── AuditLog.php
│   │   └── views/            # PHP templates (no Twig/Blade — plain PHP)
│   │       ├── layout/main.php         # Master layout (navbar, CSS vars, Montserrat)
│   │       ├── auth/login.php
│   │       ├── app/dashboard.php
│   │       ├── app/tickets/index.php
│   │       ├── app/tickets/show.php
│   │       ├── app/assets/index.php
│   │       ├── app/assets/show.php
│   │       ├── app/content/index.php
│   │       └── portal/dashboard.php
│   ├── migrations/
│   │   ├── 001_create_tables.php  # Full schema creation (13 tables)
│   │   └── 002_seed_data.php      # Roles, permissions, sample data
│   ├── install/
│   │   └── index.php         # Web installer wizard (deletes itself after success)
│   └── storage/              # File uploads — outside web root, .htaccess denies direct access
├── README.md                 # Project documentation and deployment guide
└── LICENSE                   # MIT
```

---

## Technology Stack

| Layer | Technology |
|---|---|
| Language | PHP 8.2+ |
| Database | MySQL 8 (PDO, utf8mb4) |
| Template engine | Plain PHP (output buffering) |
| External dependency | PHPMailer ^6.9 |
| Web server | Apache (via `.htaccess`) |
| Autoloading | PSR-4 via custom `spl_autoload_register` (no Composer autoloader at runtime) |
| Frontend | Plain HTML + CSS custom properties, Montserrat from Google Fonts, no JS framework |

There is **no build step**, **no Node.js**, **no TypeScript**, and **no test suite**.

---

## Environment Configuration

The app reads from a `.env` file in `sacci_brand_hub/`. Environment variables are loaded by `Config\loadEnv()` and accessed via `Config\env($key, $default)`.

Required environment variables:

```ini
APP_NAME=Sacci Brand Hub
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=sacci_brand_hub
DB_USERNAME=your_db_user
DB_PASSWORD=your_db_password
```

Optional (SMTP for PHPMailer):

```ini
MAIL_HOST=smtp.example.com
MAIL_PORT=587
MAIL_USERNAME=user@example.com
MAIL_PASSWORD=secret
MAIL_FROM=noreply@houseofsacci.com
```

During development, copy `.env.example` (if present) or create `.env` manually. The web installer (`install/index.php`) can write this file via the browser.

---

## Local Development Setup

No package manager scripts exist. Setup is manual:

```bash
# 1. Install PHP dependencies
cd sacci_brand_hub
composer install

# 2. Create .env file with DB credentials (see above)
cp .env.example .env   # then edit values

# 3. Run migrations (requires DB to already exist)
php migrations/001_create_tables.php
php migrations/002_seed_data.php

# 4. Start a dev server (PHP built-in server)
php -S localhost:8000 -t sacci_brand_hub sacci_brand_hub/index.php
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/therealjohndough) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
