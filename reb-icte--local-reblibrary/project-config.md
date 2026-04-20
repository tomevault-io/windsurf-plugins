---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Plugin Overview

**REB Library** (`local_reblibrary`) is a Moodle local plugin for managing the Rwanda Education Board digital library. It provides a complete resource management system with categorization, author management, and integration with Rwanda's education structure (levels, sublevels, classes, and A-Level sections).

**Location:** `moodle_app/public/local/reblibrary/`

**Version:** 1.3.1 (2025102403)

**Tech Stack:**
- Backend: PHP 8.2+ (Moodle external API / web services)
- Frontend: TypeScript + Preact + Tailwind CSS v4
- Build: Vite (compiles TypeScript to AMD modules)
- Database: MariaDB 11.4 (8 custom tables)

## Architecture Overview

### Database Schema

The plugin defines 8 custom tables in `db/install.xml`:

**Education Structure Tables:**
- `local_reblibrary_edu_levels` - Main levels (Pre-primary, Primary, Secondary)
- `local_reblibrary_edu_sublevels` - Sublevels (Nursery, Lower Primary, A Level, etc.)
- `local_reblibrary_classes` - Grade levels (N1, P4, S6, etc.) with unique class codes
- `local_reblibrary_sections` - A-Level subject combinations (PCM, HEG, etc.)

**Resource Management Tables:**
- `local_reblibrary_resources` - Main resource/book table (title, ISBN, description, file URLs)
- `local_reblibrary_authors` - Author information (first name, last name, bio)
- `local_reblibrary_categories` - Hierarchical categories (self-referencing for nesting)

**Junction Tables:**
- `local_reblibrary_res_assigns` - Links resources to classes/sections
- `local_reblibrary_res_categories` - Many-to-many resources ↔ categories

### Web Services API

All CRUD operations are exposed via Moodle external API (AJAX-enabled). Defined in `db/services.php`:

**Resources API** (`classes/external/resources.php`):
- `local_reblibrary_get_all_resources`
- `local_reblibrary_get_resource_by_id`
- `local_reblibrary_create_resource`
- `local_reblibrary_update_resource`
- `local_reblibrary_delete_resource`

**Education Structure API** (`classes/external/edu_structure.php`):
- Level, sublevel, class, and section CRUD operations

**Categories API** (`classes/external/categories.php`):
- Category CRUD operations

**Authors API** (`classes/external/authors.php`):
- Author CRUD operations

### Frontend Architecture

**TypeScript + Preact + Tailwind CSS:**
- Source: `amd/src/*.ts` and `amd/src/*.tsx`
- Components: `amd/src/components/` (admin UI, education structure, shared components)
- Build output: `amd/build/*.js` (AMD modules for Moodle's RequireJS)

**Key Frontend Modules:**
- `library-home.ts` - Public library home page
- `dashboard.ts` - Admin dashboard
- `resources.ts` - Resource management UI
- `categories.ts` - Category management UI
- `ed-structure.ts` - Education structure management UI

**Build System:**
- Vite configuration: `vite.config.ts`
- Auto-discovers all root-level `.ts`/`.tsx` files in `amd/src/`
- Excludes utility files: `app`, `types`, `store` (imported by other modules)
- Compiles to AMD format with bundled dependencies (Preact, Tailwind CSS)
- CSS is injected via JavaScript (no separate CSS files needed)

### Admin Interface

Admin pages located in `admin/`:
- `admin/index.php` - Main dashboard
- `admin/resources.php` - Resource management
- `admin/categories.php` - Category management
- `admin/ed_structure.php` - Education structure setup

Access requires `moodle/site:config` capability.

### Capabilities

Defined in `db/access.php`:
- `local/reblibrary:view` - View library (all users)
- `local/reblibrary:manageresources` - Manage resources (teachers, managers)
- `local/reblibrary:manageauthors` - Manage authors (teachers, managers)
- `local/reblibrary:managecategories` - Manage categories (teachers, managers)
- `local/reblibrary:manageedulevels` - Manage education levels (managers only)
- `local/reblibrary:manageedusublevels` - Manage sublevels (managers only)
- `local/reblibrary:manageclasses` - Manage classes (managers only)
- `local/reblibrary:managesections` - Manage A-Level sections (managers only)
- `local/reblibrary:manageassignments` - Assign resources to classes/sections

## Development Commands

### Frontend Build (TypeScript/Preact)

All commands run from the plugin directory:

```bash
# Install dependencies (first time only)
pnpm install

# Build once
pnpm run build

# Watch for changes (development mode)
pnpm run dev
```

**Critical:** After building frontend, always purge Moodle caches:

```bash
# From project root
docker compose exec php php /var/www/html/moodle_app/admin/cli/purge_caches.php
```

### Backend Development

```bash
# Purge caches (after PHP, template, or language string changes)
docker compose exec php php /var/www/html/moodle_app/admin/cli/purge_caches.php

# Run database upgrade (after changing db/install.xml or db/upgrade.php)
docker compose exec php php /var/www/html/moodle_app/admin/cli/upgrade.php --non-interactive

# Check plugin status
docker compose exec -T mariadb mariadb -u moodleuser -pmoodlepass moodle -e \
  "SELECT name, value FROM mdl_config_plugins WHERE plugin='local_reblibrary';"
```

### Database Inspection

```bash
# Access database shell

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/REB-ICTE) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
