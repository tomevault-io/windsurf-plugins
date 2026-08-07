---
trigger: always_on
description: <laravel-boost-guidelines>
---

<laravel-boost-guidelines>
=== .ai/tooling rules ===

# Project Tooling

These conventions are specific to this project and override default package guidance.

## Testing — Pest (not PHPUnit)

- Tests use Pest. Write tests with `test()` / `it()` and `expect()`, not PHPUnit classes.
- `tests/Pest.php` binds `Tests\TestCase` and `RefreshDatabase` to the `Feature/` and `Unit/` directories, so individual test files do not need `extends TestCase` or `use RefreshDatabase`.
- Use `beforeEach()` instead of `setUp()`, and file-scoped `function` definitions instead of private helper methods.
- Create a test with `php artisan make:test {name}` then convert it to Pest functional style, or write the Pest file directly.
- Run: `composer test`, `./vendor/bin/pest --compact`, or filter with `./vendor/bin/pest --filter=name`.
- Do NOT convert Pest tests to PHPUnit.

## JS package manager — bun (not npm)

- Use `bun install`, `bun run <script>`, and `bunx <bin>`. Do not use `npm`, `npx`, or `pnpm`.
- The lockfile is `bun.lock`.

## JS lint — oxlint (not eslint)

- Config: `.oxlintrc.json` (type-aware linting enabled via `oxlint-tsgolint`).
- Run `bun run lint` (auto-fix) or `bun run lint:check`.

## JS format — oxfmt (not prettier)

- Config: `.oxfmtrc.json`, which includes Tailwind CSS class sorting and import sorting.
- Run `bun run format` (write) or `bun run format:check`.

## PHP refactoring — Rector

- Config: `rector.php` (PHP sets + Laravel sets via `driftingly/rector-laravel`).
- Run `composer refactor:check` (dry-run) to preview, then `composer refactor` to apply.
- Review Rector diffs before committing; do not blanket-apply.

## PHP style + static analysis (unchanged)

- Pint for code style: `vendor/bin/pint --dirty` (run before finalizing PHP changes).
- Larastan for static analysis: `composer types:check` (level 7).

## Full local gate

`composer ci:check` runs oxlint, oxfmt, tsc, Pint, Larastan, and the Pest suite.

=== foundation rules ===

# Laravel Boost Guidelines

The Laravel Boost guidelines are specifically curated by Laravel maintainers for this application. These guidelines should be followed closely to ensure the best experience when building Laravel applications.

## Foundational Context

This application is a Laravel application and its main Laravel ecosystems package & versions are below. You are an expert with them all. Ensure you abide by these specific packages & versions.

- php - 8.5
- inertiajs/inertia-laravel (INERTIA_LARAVEL) - v3
- laravel/fortify (FORTIFY) - v1
- laravel/framework (LARAVEL) - v13
- laravel/prompts (PROMPTS) - v0
- laravel/wayfinder (WAYFINDER) - v0
- larastan/larastan (LARASTAN) - v3
- laravel/boost (BOOST) - v2
- laravel/mcp (MCP) - v0
- laravel/pail (PAIL) - v1
- laravel/pint (PINT) - v1
- laravel/sail (SAIL) - v1
- pestphp/pest (PEST) - v4
- phpunit/phpunit (PHPUNIT) - v12
- rector/rector (RECTOR) - v2
- @inertiajs/react (INERTIA_REACT) - v3
- react (REACT) - v19
- tailwindcss (TAILWINDCSS) - v4
- @laravel/vite-plugin-wayfinder (WAYFINDER_VITE) - v0

## Skills Activation

This project has domain-specific skills available in `**/skills/**`. You MUST activate the relevant skill whenever you work in that domain—don't wait until you're stuck.

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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coollabsio/shoutrrr](https://github.com/coollabsio/shoutrrr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
