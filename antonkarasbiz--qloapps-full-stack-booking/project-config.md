---
trigger: always_on
description: QloApps is an open-source hotel reservation and property management platform. It enables hotels to manage rooms, bookings, guests, and payments through a web-based system.
---

# AGENTS.md

## Project Overview

QloApps is an open-source hotel reservation and property management platform. It enables hotels to manage rooms, bookings, guests, and payments through a web-based system.

## Purpose of this File

This document provides guidance for AI coding agents contributing to the QloApps project. It defines conventions, safety rules, and workflows to ensure consistent and secure code generation.

## Technology Stack

- **Language:** PHP 8.1–8.4 (backend), Smarty 3.x (templates), JavaScript/jQuery (frontend)
- **Database:** MySQL 5.7, 8.0+; MariaDB 10.5, 10.6, 10.11, 11.0, 11.2, 11.4
- **Architecture:** MVC with hook-based module system
- **License:** OSL-3.0 (core), AFL-3.0 (modules)
- **Required PHP Extensions:** PDO_MySQL, cURL, OpenSSL, SOAP, GD, SimpleXML, DOM, Zip, Phar

## Environment Setup

Install dependencies:
```bash
composer install
```

Clear caches:
```bash
rm -rf cache/smarty/compile/* cache/smarty/cache/*
rm -f cache/class_index.php
```

Clear class cache after adding or modifying overrides.

## Project Structure

```
.
├── classes/              # Core models extending ObjectModel
├── controllers/          # Front and admin controllers
│   ├── admin/
│   └── front/
├── modules/              # Feature modules with isolated functionality
├── override/             # Core class and controller overrides
├── themes/               # Smarty templates (.tpl files)
├── config/               # Configuration files (settings.inc.php contains secrets)
├── cache/                # Generated cache files
├── tests/                # PHPUnit test suite
```

**QloApps Agent Skills:** Install reusable development skills using:
```bash
npx skills add Qloapps/agent-skills
```
Available skills: module-development, payment-module-development, stats-module-development. Check installed skills before implementing new functionality.

## Architecture Overview

**MVC Pattern**
- Models: classes/ extending ObjectModel
- Controllers: FrontController or AdminController
- Views: Smarty templates (.tpl)

**Modules**
- Located in modules/<modulename>/
- Provide isolated functionality
- Integrate using hooks

**Overrides**
- Located in override/
- Extend core classes using the Core suffix
- Clear class cache after creating overrides

**Context**
- Access runtime objects using Context::getContext()

## Core vs Module Development Rules

**When working on a module:**
1. Never modify core files directly
2. Use **hooks** to integrate module functionality into core features
3. If no suitable hook exists, create a **custom hook** — but only if the hook placement is generic and useful for other modules too
4. Use **overrides** only as a last resort — overrides can conflict with other module override files and require manual resolution

**When working on a core feature:**
- Make changes directly in core files
- Do not use hooks or overrides for core-to-core changes

## Coding Conventions

**Classes:** PascalCase — `HotelBookingData`
**Methods:** camelCase — `getBookingDetails()`
**Variables:** camelCase — `$hotelId`
**Constants:** UPPER_SNAKE_CASE — `BOOKING_STATUS_CONFIRMED`
**Database Tables:** _DB_PREFIX_ + lowercase_snake — `qlo_hotel_booking`
**Config Keys:** MODULENAME_SETTING — `HOTELRESERVATION_ENABLED`
**Files:** One class per file, filename matches class name
**Templates:** lowercase-hyphens.tpl — `booking-form.tpl`

Add PHPDoc blocks to all classes and methods.

## Translation

Never hardcode user-facing English strings — always wrap them in the appropriate translation method.

| Context | Method |
|---------|--------|
| Module main file | `$this->l('string')` |
| Module admin controller | `$this->l('string')` |
| Module front controller | `$this->module->l('string', 'controllerName')` |
| Module classes | `$objModule->l('string', 'ClassName')` |
| Core admin controller | `$this->l('string')` |
| Core front controller | `Tools::displayError('string')` |
| Smarty template (core) | `{l s='string'}` |
| Smarty template (module) | `{l s='string' mod='modulename'}` |

## Multi-language

- Always include `id_lang` in queries that return translatable content
- Use `Context::getContext()->language->id` for the current language

## Module Development Guidelines

**Module Location:** modules/<modulename>/

**Required Files:**
- <modulename>.php — Main class extending Module
- config.xml — Module metadata

**Optional Directories:**
- classes/ — Module-specific models
- controllers/ — Module controllers
- views/templates/ — Smarty templates
- upgrade/ — Version migration scripts

**Hook Integration:**
- Register hooks in install() method
- Unregister hooks in uninstall() method
- Keep hook handlers lightweight

**Configuration:**
- Store settings using Configuration::updateValue()
- Retrieve settings using Configuration::get()
- Prefix config keys with module name

## Database Rules

**Table Prefix:** Always use _DB_PREFIX_ constant instead of hardcoding the table prefix

**Escaping:**
- Strings: pSQL($value)
- Integers: (int)$value
- Table/column names: bqSQL($name)

**Preferred Access:** Use ObjectModel for CRUD operations instead of raw SQL queries


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antonkarasbiz/QloApps_Full-stack_Booking](https://github.com/antonkarasbiz/QloApps_Full-stack_Booking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
