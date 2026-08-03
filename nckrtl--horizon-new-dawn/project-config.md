---
trigger: always_on
description: Horizon New Dawn is a Laravel 12 and 13 package that replaces Laravel Horizon's bundled UI with a package-owned React and Inertia interface.
---

# Horizon New Dawn

Horizon New Dawn is a Laravel 12 and 13 package that replaces Laravel Horizon's bundled UI with a package-owned React and Inertia interface.

## Project Context

- Read `README.md` before making changes to understand the package's purpose, supported versions, installation flow, and current feature scope.
- Read `docs/architecture.md` before making architectural changes to understand route ownership, backend data flow, frontend isolation, asset delivery, and extension boundaries.

## Versions

- Support only PHP 8.3+, Laravel 12.38+ or 13, Horizon 5.46.0+, Inertia 3, React 19, Tailwind 4, and Wayfinder 0.1.
- Do not add compatibility branches for older major versions.
- Use Bun for frontend dependencies and scripts.

## PHP

- Begin every PHP file with `declare(strict_types=1);`.
- Follow Laravel conventions and keep controllers thin.
- Use Spatie Laravel Data for structured page props.
- Write Pest tests before production code and verify the expected failure.

## Frontend

- Compose interfaces from shadcn/ui components installed through its CLI.
- Use semantic theme tokens; do not add raw status colors or manual dark-mode overrides.
- Treat `resources/js/generated` as Wayfinder-generated code. Never edit it manually.
- Resolve custom Horizon paths through the package route helper.

## Verification

```bash
composer quality
bun run test
bun run typecheck
bun run build
```

===

<laravel-boost-guidelines>
=== .ai/package-workbench rules ===

# Package Workbench

- This is a Laravel package repository with Orchestra Testbench and no root `artisan` executable. Run Artisan commands through `php vendor/bin/testbench`.
- When running Laravel Boost commands directly, use `APP_BASE_PATH=. APP_ENV=local VIEW_COMPILED_PATH=bootstrap/cache php vendor/bin/testbench boost:<command>` so Boost scans this package instead of Testbench's internal application skeleton.

=== foundation rules ===

# Laravel Boost Guidelines

The Laravel Boost guidelines are specifically curated by Laravel maintainers for this application. These guidelines should be followed closely to ensure the best experience when building Laravel applications.

## Foundational Context

This application is a Laravel application and its main Laravel ecosystems package & versions are below. You are an expert with them all. Ensure you abide by these specific packages & versions.

- php - 8.5
- inertiajs/inertia-laravel (INERTIA_LARAVEL) - v3
- laravel/framework (LARAVEL) - v13
- laravel/horizon (HORIZON) - v5
- laravel/prompts (PROMPTS) - v0
- larastan/larastan (LARASTAN) - v3
- laravel/boost (BOOST) - v2
- laravel/mcp (MCP) - v0
- laravel/pail (PAIL) - v1
- laravel/pint (PINT) - v1
- laravel/wayfinder (WAYFINDER) - v0
- pestphp/pest (PEST) - v4
- phpunit/phpunit (PHPUNIT) - v12
- @inertiajs/react (INERTIA_REACT) - v3
- react (REACT) - v19
- @laravel/vite-plugin-wayfinder (WAYFINDER_VITE) - v0
- tailwindcss (TAILWINDCSS) - v4

## Conventions

- You must follow all existing code conventions used in this application. When creating or editing a file, check sibling files for the correct structure, approach, and naming.
- Use descriptive names for variables and methods. For example, `isRegisteredForDiscounts`, not `discount()`.
- Check for existing components to reuse before writing a new one.

## Verification Scripts

- Do not create verification scripts or tinker when tests cover that functionality and prove they work. Unit and feature tests are more important.

## Application Structure & Architecture

- Stick to existing directory structure; don't create new base folders without approval.
- Do not change the application's dependencies without approval.

## Frontend Bundling

- If the user doesn't see a frontend change reflected in the UI, it could mean they need to run `bun run build`, `bun run dev`, or `composer run dev`. Ask them.

## Documentation Files

- You must only create documentation files if explicitly requested by the user.

## Replies

- Be concise in your explanations - focus on what's important rather than explaining obvious details.

=== boost rules ===

# Laravel Boost

## Tools

- Laravel Boost is an MCP server with tools designed specifically for this application. Prefer Boost tools over manual alternatives like shell commands or file reads.
- Use `database-query` to run read-only queries against the database instead of writing raw SQL in tinker.
- Use `database-schema` to inspect table structure before writing migrations or models.
- Use `get-absolute-url` to resolve the correct scheme, domain, and port for project URLs. Always use this before sharing a URL with the user.
- Use `browser-logs` to read browser logs, errors, and exceptions. Only recent logs are useful, ignore old entries.

## Searching Documentation (IMPORTANT)

- Always use `search-docs` before making code changes. Do not skip this step. It returns version-specific docs based on installed packages automatically.
- Pass a `packages` array to scope results when you know which packages are relevant.
- Use multiple broad, topic-based queries: `['rate limiting', 'routing rate limiting', 'routing']`. Expect the most relevant results first.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nckrtl/horizon-new-dawn](https://github.com/nckrtl/horizon-new-dawn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
