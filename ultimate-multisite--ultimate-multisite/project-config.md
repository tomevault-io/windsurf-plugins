---
trigger: always_on
description: **Ultimate Multisite** is the user-facing product name. Use "Ultimate Multisite" in all
---

# AGENTS.md — Ultimate Multisite

**Ultimate Multisite** is the user-facing product name. Use "Ultimate Multisite" in all
UI text, docs, and user-facing strings. The code namespace `WP_Ultimo` and the `wu_`
function/hook prefix are preserved for backwards compatibility — do not rename them.

WordPress Multisite WaaS plugin (formerly WP Ultimo). PHP 7.4+, WP 5.3+, GPL v2.
Root namespace: `WP_Ultimo`. Text domain: `ultimate-multisite`.

## Build / Test / Lint Commands

```bash
# Install dependencies
composer install && npm install

# Run full test suite (requires WP test environment — see bin/install-wp-tests.sh)
vendor/bin/phpunit

# Run a single test class
vendor/bin/phpunit --filter Cart_Test

# Run a single test method
vendor/bin/phpunit --filter test_constructor_initializes_defaults

# Run tests with coverage
php -d zend_extension=xdebug -d xdebug.mode=coverage vendor/bin/phpunit \
  --coverage-html=coverage-html --coverage-clover=coverage.xml

# Lint PHP (PHPCS — WordPress coding standards)
vendor/bin/phpcs
vendor/bin/phpcbf                    # auto-fix

# Lint a single PHP file
vendor/bin/phpcs inc/path/to/file.php
vendor/bin/phpcbf inc/path/to/file.php

# Static analysis (PHPStan level 0)
vendor/bin/phpstan analyse

# Lint JS / CSS
npm run lint:js
npm run lint:css

# All quality checks
npm run check                        # lint + stan + test
```

## Project Structure

```text
ultimate-multisite.php   # Plugin entry point, defines WP_ULTIMO_PLUGIN_FILE
constants.php            # Plugin constants and feature flags
sunrise.php              # MU-plugin for domain mapping (loaded before WP)
inc/
  models/                # Data models (Base_Model subclasses)
  managers/              # Singleton managers (business logic, hooks)
  database/              # BerlinDB tables, schemas, queries, enums
  gateways/              # Payment gateways (Stripe, PayPal, Manual, Free)
  checkout/              # Cart, Checkout, Line_Item, signup fields
  admin-pages/           # WP admin page controllers (full admin pages)
  admin/                 # WP admin utilities (config checker, network columns)
  list-tables/           # WP_List_Table subclasses
  integrations/          # Host provider integrations (cPanel, Cloudflare, etc.)
  functions/             # Procedural helper functions (wu_get_*, wu_create_*)
  sso/                   # Single sign-on across subsites
  helpers/               # Utility classes (Arr, Hash, Validator, etc.)
  apis/                  # REST API, WP-CLI, MCP traits
  ui/                    # Frontend elements and shortcodes
  traits/                # Shared traits (Singleton, deprecated compat)
  exception/             # Runtime_Exception
  builders/              # Block editor / Gutenberg field builders
  compat/                # Third-party plugin compatibility (Elementor, WooCommerce, etc.)
  deprecated/            # Deprecated functions and classes (backward compat)
  domain-mapping/        # Domain mapping helpers (class-helper.php, class-primary-domain.php)
  duplication/           # Site duplication utilities
  external-cron/         # External cron job scheduling and management
  installers/            # Plugin installation, migration, and setup utilities
  invoices/              # Invoice generation (class-invoice.php)
  limitations/           # Per-feature plan limit classes (class-limit-*.php)
  limits/                # Plugin/site-level limits enforcement
  loaders/               # Table and asset loaders
  mercator/              # Mercator subdomain mapping plugin integration
  objects/               # Value objects (billing address, note, visits)
  site-exporter/         # Site export tools
  site-templates/        # Site template management (class-template-placeholders.php)
  tax/                   # Tax calculation and dashboard management
  template-library/      # Template library (API client, installer, repository)
  country/               # Country-specific tax/address configuration classes
  contracts/             # PHP contracts (e.g. contracts/Session.php)
  interfaces/            # Interface definitions (e.g. interface-singleton.php)
  debug/                 # Debug utilities (class-debug.php)
  development/           # Development toolkit and Query Monitor integration (dev-only)
tests/
  WP_Ultimo/             # Unit tests mirroring inc/ structure (main test suite)
  Admin_Pages/           # Admin page tests
  Builders/              # Block editor / widget builder tests
  functional/            # Functional/integration tests (e.g. SSO)
  unit/                  # Standalone unit tests (API schema, checkout request, etc.)
  e2e/                   # Cypress E2E tests
  bootstrap.php          # WP test bootstrap (loads plugin via muplugins_loaded)
views/                   # PHP template files
assets/                  # JS, CSS, images, fonts
```

## Code Style

### PHP (WordPress Coding Standards via PHPCS)

- **Indentation**: Tabs (not spaces). Tab width = 4.
- **Braces**: Opening brace on same line as declaration (`class Foo {`, `if (...) {`).
- **Arrays**: Short syntax `[]` allowed. No spaces inside brackets: `['key' => 'val']`.
- **Ternary**: Short ternary `?:` allowed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ultimate-Multisite/ultimate-multisite](https://github.com/Ultimate-Multisite/ultimate-multisite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
