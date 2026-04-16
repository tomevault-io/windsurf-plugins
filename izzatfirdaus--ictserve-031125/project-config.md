---
trigger: always_on
description: This document provides specific instructions and context for Gemini when working within this project.
---

# Gemini Workspace Instructions

This document provides specific instructions and context for Gemini when working within this project.

## Project Overview

This project is a Laravel application named "ICTServe". It uses Filament for the admin panel, Livewire for dynamic interfaces, and Volt for single-file Livewire components. The application includes a helpdesk and loan application modules.

**Main Technologies:**

- PHP 8.4.11
- Laravel 12.38.1
- Filament 4.1.10
- Livewire 3.6.4
- Volt 1.9.0
- Database: sqlite

## Gemini's Role

Your role is to act as a full-stack developer, assisting with tasks ranging from frontend to backend development. You should be able to:

- Write and refactor code in PHP, JavaScript, and other relevant languages.
- Create and modify Laravel components such as controllers, models, and views.
- Work with Filament to build and maintain the admin panel.
- Develop and enhance Livewire components.
- Write and run tests to ensure code quality.
- Help with debugging and troubleshooting.

## Core Mandates & Guidelines

- **Adhere to Conventions:** Rigorously follow existing project conventions (coding style, naming, etc.). Analyze surrounding code before making changes.
- **Library Usage:** Do not introduce new libraries or frameworks without verifying their established use in the project. Check `package.json`, `composer.json`, `requirements.txt`, etc.
- **Code Style:** Mimic the style, structure, and architectural patterns of the existing codebase.
- **Testing:** When adding features or fixing bugs, include corresponding tests to maintain quality.
- **Confirmation:** If a request is ambiguous or requires a significant change, ask for clarification before proceeding.

## Getting Started

1. **Review the Codebase:** Before making any changes, take time to understand the relevant parts of the project. Use the available tools to explore the code.
2. **Run Existing Tests:** Familiarize yourself with the testing process by running the project's test suite.
3. **Start with Small Tasks:** Begin with smaller, well-defined tasks to get a feel for the workflow.

## Commands & Tools

- **Build:** `npm run build`
- **Test:** `composer test`
- **Lint:** `composer lint`
- **Other:**
  - `npm run test:e2e`: Run Playwright end-to-end tests.
  - `vendor/bin/pint`: Run Laravel Pint to fix code style issues.
  - `vendor/bin/phpstan analyse`: Run PHPStan for static analysis.
  - `vendor/bin/phpinsights`: Run PHP Insights for code quality analysis.

## Important File Locations

- **Source Code:** `app/`
- **Tests:** `tests/`
- **Configuration:** `config/`
- **Documentation:** `docs/`
- **Filament Admin Panel:** `app/Filament/`
- **Livewire Components:** `app/Livewire/`
- **Routes:** `routes/`
- **Database Migrations:** `database/migrations/`

## Additional Notes

### Installed Packages
<details>
<summary>View Packages</summary>

| Roster Name   | Version  | Package Name            |
|---------------|----------|-------------------------|
| FILAMENT      | 4.1.10   | filament/filament       |
| LARAVEL       | 12.38.1  | laravel/framework       |
| PROMPTS       | 0.3.7    | laravel/prompts         |
| REVERB        | 1.6.1    | laravel/reverb          |
| LIVEWIRE      | 3.6.4    | livewire/livewire       |
| VOLT          | 1.9.0    | livewire/volt           |
| LARASTAN      | 3.8.0    | larastan/larastan       |
| BREEZE        | 2.3.8    | laravel/breeze          |
| MCP           | 0.3.3    | laravel/mcp             |
| PINT          | 1.25.1   | laravel/pint            |
| SAIL          | 1.48.0   | laravel/sail            |
| PHPUNIT       | 11.5.44  | phpunit/phpunit         |
| ECHO          | 2.2.6    | laravel-echo            |
| TAILWINDCSS   | 3.4.18   | tailwindcss             |

</details>

### Eloquent Models
<details>
<summary>View Models</summary>

- `App\Models\HelpdeskComment`
- `App\Models\HelpdeskTicket`
- `App\Models\LoanApplication`
- `App\Models\User`

</details>

===

<laravel-boost-guidelines>
=== foundation rules ===

# Laravel Boost Guidelines

The Laravel Boost guidelines are specifically curated by Laravel maintainers for this application. These guidelines should be followed closely to enhance the user's satisfaction building Laravel applications.

## Foundational Context
This application is a Laravel application and its main Laravel ecosystems package & versions are below. You are an expert with them all. Ensure you abide by these specific packages & versions.

- php - 8.4.1
- filament/filament (FILAMENT) - v4
- laravel/framework (LARAVEL) - v12
- laravel/mcp (MCP) - v0
- laravel/prompts (PROMPTS) - v0
- laravel/pulse (PULSE) - v1
- laravel/reverb (REVERB) - v1
- laravel/sanctum (SANCTUM) - v4
- laravel/socialite (SOCIALITE) - v5
- livewire/livewire (LIVEWIRE) - v3
- livewire/volt (VOLT) - v1
- larastan/larastan (LARASTAN) - v3
- laravel/breeze (BREEZE) - v2
- laravel/pint (PINT) - v1
- laravel/telescope (TELESCOPE) - v5
- phpunit/phpunit (PHPUNIT) - v11
- laravel-echo (ECHO) - v2
- tailwindcss (TAILWINDCSS) - v4

## Conventions
- You must follow all existing code conventions used in this application. When creating or editing a file, check sibling files for the correct structure, approach, naming.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IzzatFirdaus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
