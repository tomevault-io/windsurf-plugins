---
trigger: always_on
description: Lens for Laravel is a local-first accessibility auditing package for Laravel applications. It renders application pages in Chromium, runs axe-core, maps violations back to Blade/React/Vue source files, and exposes the results through a dashboard and the `lens:audit` Artisan command.
---

# Lens for Laravel - Contributor Guide

## Project Scope

Lens for Laravel is a local-first accessibility auditing package for Laravel applications. It renders application pages in Chromium, runs axe-core, maps violations back to Blade/React/Vue source files, and exposes the results through a dashboard and the `lens:audit` Artisan command.

The current development line is v3.5.0. Its new features are an optional explicit AI model override and dashboard info about the active AI Fix model. The released v3.4.0 added authenticated scans for pages behind login and four additional AI Fix providers (OpenRouter, xAI, DeepSeek, Mistral). New compatibility, WCAG selection, reliability, localization, source-mapping, and documentation work in this branch must otherwise be described as v3 functionality. Keep v2.0/v2.1 and incremental v3.0/v3.1/v3.2/v3.3/v3.4 upgrade notes as historical documentation.

The package supports PHP 8.2+ and Laravel 10-13 for its core, non-AI features.

AI Fix is an optional integration with a narrower compatibility range:

- PHP 8.3+
- Laravel 12+
- optional `laravel/ai` Composer package

Do not add `laravel/ai` back to production `require`. Core installation on PHP 8.2 and Laravel 10/11 must remain possible. Code that touches the AI SDK must be guarded through `AiFixAvailability`.

## Main Components

- `src/Services/AxeScanner.php` - browser-based axe-core scans, including interactive states
- `src/Services/SiteCrawler.php` - sitemap and internal-link discovery
- `src/Services/HttpsClientConfiguration.php` - shared TLS behavior for HTTP and Chromium clients
- `src/Services/FileLocator.php` - heuristic Blade/React/Vue source mapping
- `src/Services/InteractionScriptParser.php` - state script validation and parsing
- `src/Services/BaselineManager.php` - stable CI baseline fingerprints
- `src/Services/ScanComparator.php` - URL- and state-aware history comparison
- `src/Support/UrlNormalizer.php` - host-independent URL path/query identity
- `src/Http/Middleware/SetLensLocale.php` - configured and session-selected package locale
- `src/Services/AiFixAvailability.php` - runtime and optional-SDK capability checks
- `src/Ai/AccessibilityFixAgent.php` - structured v3 AI agent options and provider-specific controls
- `src/Services/AiFixContextExtractor.php` - bounded semantic element/component extraction
- `src/Services/AiFixPromptRunner.php` - optional Laravel AI SDK boundary and response metadata
- `src/Services/AiFixer.php` - prompt orchestration, controlled retry, safe errors, and diagnostics
- `src/DTOs/AuthenticatedScanContext.php` - server-issued session cookies for authenticated scans
- `src/Services/AuthenticatedScanResolver.php` - guard login, allowlist, and auth-state restore
- `src/Support/Wcag.php` - supported WCAG versions, cumulative axe-core tags, and result-level classification
- `src/Console/Commands/LensAuditCommand.php` - CLI audit workflow
- `routes/web.php` - dashboard JSON endpoints
- `resources/views/dashboard.blade.php` - dashboard interface
- `resources/views/state-recorder.blade.php` - visual interaction recorder
- `resources/views/report.blade.php` - PDF report

## Compatibility Rules

- Keep core code syntactically compatible with PHP 8.2.
- Keep core dependencies compatible with Laravel 10, 11, 12, and 13.
- Treat AI Fix as unavailable when the runtime or optional SDK is unsupported.
- Never let a missing AI SDK break scanning, crawling, history, PDF reports, previews, interactive states, or the CLI.
- Keep WCAG 2.0 as the default unless a breaking release explicitly changes it. WCAG 2.1 and 2.2 scans must include the earlier cumulative rule tags.
- Treat the WCAG version and conformance level as separate controls: `--wcag` selects 2.0/2.1/2.2, while `--a`, `--aa`, and `--all` filter result levels.
- Preserve URL path and query in history/baseline identity while ignoring scheme and host, and preserve interactive state labels as a separate identity dimension.
- Keep the default CLI `--threshold` at `0` and the default crawler limit at `50` unless a breaking release explicitly changes them.
- Keep every compatibility surface, including the dashboard footer, aligned with core Laravel 10, 11, 12, and 13 support.
- When compatibility changes, update `composer.json`, `README.md`, `CONTEXT.md`, tests, and the separate `lens-for-laravel-website` documentation together.

## Composer Metadata

- The package root is a Composer `library` named `webcrafts-studio/lens-for-laravel`; never describe it as a Laravel skeleton application.
- Do not add a hard-coded package version to `composer.json`; releases are determined from Git tags.
- Keep `laravel/ai` optional under `suggest`, never under production `require`.
- Keep the package description, keywords, homepage, support URLs, PHP/Laravel constraints, README, and public website consistent.
- The separate website is a Laravel `project` named `webcrafts-studio/laravel-lens-website`; its Composer metadata must describe the documentation website rather than the Laravel skeleton.

## Development Workflow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [webcrafts-studio/lens-for-laravel](https://github.com/webcrafts-studio/lens-for-laravel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
