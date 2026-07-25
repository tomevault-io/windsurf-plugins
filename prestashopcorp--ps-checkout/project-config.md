---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## About

PrestaShop Checkout is the official PrestaShop payment module in partnership with PayPal. The `main` branch supports v5 for PrestaShop 1.7, 8, and 9 simultaneously. Older versions use dedicated maintenance branches (`prestashop/9.x`, `prestashop/8.x`, `prestashop/1.7.x`).

## Development Setup

Requirements: PHP, Composer, Docker, Docker Compose, GNU Make.

```bash
cp .env.dist .env          # Configure PS_VERSION_TAG, MODULE_VERSION, etc.
cp <MODULE_VERSION>/.env.dist <MODULE_VERSION>/.env
cp docker-compose.local.yml.dist docker-compose.local.yml
make build                 # Build Docker images
make up                    # Start containers + install root dependencies
```

The shop runs at `http://localhost:8991` (admin: `demo@prestashop.com` / `prestashop_demo`).

Key `.env` variables:
- `MODULE_VERSION`: `ps17`, `ps8`, or `ps9` — controls which PrestaShop container to target
- `PS_VERSION_TAG`: Docker image tag for the PrestaShop version (e.g., `8`)
- `SENTRY_DSN`: Glitchtip/Sentry DSN for error monitoring

## Commands

```bash
make lint                  # php-cs-fixer + autoindex
make php-cs-fixer          # Fix code style only
make phpstan               # PHPStan static analysis (uses $MODULE_VERSION)
make phpstan-baseline      # Regenerate PHPStan baseline for current $MODULE_VERSION
make phpstan-baseline-all  # Regenerate baselines for all versions

make unit-test             # All unit tests (api, utility, core, presentation)
make integration-test      # All integration tests (module, core, infrastructure)
make test                  # Full test suite

# Run a single test suite directly
make php-unit-core
make php-unit-api
make php-unit-utility
make php-unit-presentation
make php-integration
make php-integration-core
make php-integration-infrastructure

make ssh                   # Shell into the running PrestaShop container
make install-module        # Install the module in PrestaShop via console
```

Module logs (inside project root, not container): `prestashop/<PS_VERSION_TAG>/var/logs/ps_checkout-*-<YYYY-MM-DD>`.

All `make` test commands run inside the Docker container using `$MODULE_VERSION` and `$PS_VERSION_TAG` from `.env`.
`make phpstan` runs locally without Docker. All `make *-test` commands require a running Docker container (`make up`).

Do not run `phpunit` directly from the host — cross-package autoloading (e.g., `api/` classes in `core/` tests) only resolves inside the Docker container via the module's `vendor/autoload.php`. Always use `make` commands for tests.

Code style check without fixing: `composer cs` (from root).

## Architecture

### Multi-version monorepo

The repository targets three PrestaShop versions using a monorepo approach:

```
ps17/   — PS 1.7 module shell (PHP ^7.1)
ps8/    — PS 8.x module shell (PHP ^7.2|^8.1)
ps9/    — PS 9.x module shell (PHP ^8.1, Symfony 6.4)
```

Each version directory contains:
- `ps_checkout.php` — main module entry point extending `PaymentModule`
- `sentry.php` — Sentry/Glitchtip error monitoring initialization
- `config/` — Symfony service container YAML definitions
- `tests/` — module-level integration tests
- `composer.json` — version-specific dependencies

The version-specific directories are thin shells. All business logic lives in the shared monorepo packages below.

### Shared packages (vendor/invertus/)

The monorepo packages are in the root alongside `ps17/`, `ps8/`, `ps9/` and symlinked/mounted into each version's `vendor/invertus/`:

| Package | Namespace | Responsibility |
|---|---|---|
| `api/` | `PsCheckout\Api\` | HTTP controllers, DTOs, value objects |
| `core/` | `PsCheckout\Core\` | Business logic: orders, webhooks, PayPal, settings, payment tokens |
| `infrastructure/` | `PsCheckout\Infrastructure\` | PrestaShop adapters, repositories, controllers, loggers |
| `presentation/` | `PsCheckout\Presentation\` | Presenters for front/back-office UI |
| `utility/` | `PsCheckout\Utility\` | Shared array/string/payload utilities |

The `ps<version>/src/` directory (namespace `PsCheckout\Module\`) contains only version-specific presentation overrides.

### Key domain areas in `core/`

- `PayPal/` — PayPal API integration (orders, payments, webhooks, funding sources, refunds, Apple/Google Pay)
- `Order/` + `OrderState/` — Order processing and status transitions
- `Webhook/` + `WebhookDispatcher/` — Webhook event handling pipeline
- `Settings/` — Configuration management
- `PaymentToken/` — Payment token lifecycle

### Key areas in `infrastructure/`

- `Adapter/` — Bridge between the core domain and PrestaShop's legacy APIs
- `Repository/` — Database access layer
- `Bootstrap/` — Module installation/upgrade logic
- `Controller/` — Web controllers (front + admin)
- `Environment/` — Runtime environment detection

### CI/CD (`.github/workflows/`)

- `ci.yml` — runs tests on pull requests
- `lint.yml` — runs linting checks
- `create-testing-zip.yml` — generates module ZIP for INT/PREPROD environments
- `github-release.yml` — builds and attaches per-PS-version ZIPs to GitHub Releases (prereleased + released)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PrestaShopCorp/ps_checkout](https://github.com/PrestaShopCorp/ps_checkout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
