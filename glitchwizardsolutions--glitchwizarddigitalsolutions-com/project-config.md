---
trigger: always_on
description: This is a **legacy modernization and cleanup project** for a production XAMPP-based PHP monolith. The codebase was migrated from production to:
---

# GlitchWizard Digital Solutions - AI Coding Guide

## Project Context

This is a **legacy modernization and cleanup project** for a production XAMPP-based PHP monolith. The codebase was migrated from production to:
1. **Redesign email infrastructure** - Replace current email sending mechanisms
2. **Clean up unused code** - Identify and document/remove unused files and applications
3. **Create development specs** - Document all components for future development and troubleshooting
4. **Establish version control** - Initialize Git repository at https://github.com/GlitchWizardSolutions/glitchwizarddigitalsolutions.com.git

**AI Agent Role:** Help identify unused code, document existing functionality, plan refactoring, and implement improvements while preserving working production features.

**Version Control Note:** Only the `public/` directory is in Git. The `private/` directory (containing config.php with credentials) is NOT versioned and must be set up separately in each environment.

## Architecture Overview

This is a **legacy XAMPP-based PHP monolith** serving as a multi-tenant business management portal with 12+ separate MySQL databases. The system is structured as a collection of semi-independent subsystems under a shared authentication layer.

**Known Issues:**
- Many files present but not in use or not working correctly
- Email sending mechanisms need redesign
- Lack of comprehensive documentation
- **Path resolution differences between dev and production** - local development paths don't match production paths, causing index.php and other files to fail locally

**Critical Development Constraint:**
The application works correctly in **production** but path resolution fails in **development** due to environment differences. When fixing path issues:
- ✅ USE configuration constants (defined in `private/config.php`)
- ❌ NEVER hardcode paths specific to dev environment
- ✅ ADD new constants to `private/config.php` if needed
- ❌ NEVER use `__DIR__`, `dirname(__FILE__)`, or relative paths without validation
- The goal: Make paths work in BOTH dev and production through configuration, not code changes

### Key Directories
- `private/` - Configuration files with database credentials, paths, and API keys (NEVER commit changes here)
- `public/` - Web-accessible root; contains member login, registration, and public pages
- `public/admin/` - Admin dashboard with subsystem modules (tickets, invoices, budget, projects, newsletter, blog)
- `public/client-dashboard/` - Client-facing portal for authenticated members
- `public/client-invoices/` - Payment processing and invoice viewing
- `AI-DEV/` - **AI workspace for development artifacts** (NOT in production)
  - Use for: test files, experiments, specifications, documentation, analysis
  - All AI-generated non-production files MUST go here
  - This directory does NOT exist in production
  - Files here are committed to Git for collaboration but never deployed

### Database Architecture
The system uses **5 MySQL databases** (all with same credentials):
- `glitchwizarddigi_login_db` (db_name) - Main database with accounts, invoices, clients, domains, tickets, etc.
- `glitchwizarddigi_onthego` (db_name2) - On-the-go task management
- `glitchwizarddigi_budget_2025` (db_name7) - Budget tracking system
- `glitchwizarddigi_error_handling` (db_name9) - Error logging and monitoring
- `glitchwizarddigi_envato_blog_db` (db_name12) - Blog and newsletter system
- See `private/config.php` for complete list

**Connection Pattern:** Each subsystem creates its own PDO connection to specific databases. Main connections initialized in:
- `public/assets/includes/main.php` - Login DB
- `public/admin/assets/includes/main.php` - Admin systems
- `public/client-dashboard/assets/includes/main.php` - Multiple DBs (login, budget, blog)

## Configuration System

### Config File Hierarchy
The system uses a **layered configuration approach**:

1. **Base:** `private/config.php` - Master config with 12 databases, paths, feature flags, payment gateways, SMTP settings
2. **Bootstrap files** (include config + utilities):
   - `public/assets/includes/public-config.php` - Public pages (login, registration)
   - `public/assets/includes/process-config.php` - Form processing (minimal, no page setup)
   - `public/admin/assets/includes/admin_config.php` - Admin pages
   - Each subsystem has its own `admin_config.php` (e.g., `admin/ticket_system/assets/includes/admin_config.php`)

**Critical Pattern:** Always use the appropriate bootstrap:
```php
// For public pages with full UI
include 'assets/includes/public-config.php';

// For AJAX/form handlers (no page wrapper)
include 'assets/includes/process-config.php';

// For admin pages (requires Admin role)
require 'assets/includes/admin_config.php';
```

### Path Constants (from `private/config.php`)

**CRITICAL:** The application has environment-specific path differences. Production works correctly. Development may have path resolution failures. **ALWAYS use these constants - NEVER hardcode paths.**

```php
// Base paths - calculated from config file location
private_path          // Directory containing config.php
project_path          // Parent of private/ directory

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GlitchWizardSolutions/glitchwizarddigitalsolutions.com](https://github.com/GlitchWizardSolutions/glitchwizarddigitalsolutions.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
