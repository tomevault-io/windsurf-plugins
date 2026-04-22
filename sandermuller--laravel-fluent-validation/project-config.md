---
trigger: always_on
description: <package-boost-guidelines>
---

<package-boost-guidelines>
# Package Boost Guidelines

These guidelines replace Laravel Boost's default foundation for
repositories that are **Laravel packages**, not applications. The
framing, tooling, and trade-offs differ — follow this version when
working inside a package codebase.

## Foundational Context

This codebase is a **Laravel package** distributed via Composer, not a
Laravel application. Key consequences:

- There is no `artisan`, no `app/`, no `bootstrap/`, no `routes/`, no
  `.env`, and no database by default. A Testbench-provided Laravel
  application is spun up only at test time.
- The primary artefact is the package's public API (service provider,
  facades, classes) — everything else is scaffolding.
- Downstream apps consume this package. Every public change is a
  user-facing API change governed by semver.
- `composer.json` is the source of truth for supported PHP and
  Laravel versions. Check `require.php` and `require.illuminate/*`
  before using version-specific features.

## Use `vendor/bin/testbench`, not `php artisan`

Running artisan commands directly against the package fails — there is
no host application. Use Testbench's binary:

| Instead of | Use |
|---|---|
| `php artisan test` | `vendor/bin/pest` (or `vendor/bin/phpunit`) |
| `php artisan tinker` | `vendor/bin/testbench tinker` |
| `php artisan make:*` | Create files manually under `src/` |
| `php artisan boost:install` | `vendor/bin/testbench boost:install` |
| `php artisan boost:mcp` | `vendor/bin/testbench boost:mcp` |
| `php artisan vendor:publish` | `vendor/bin/testbench vendor:publish` |

Register the package's service provider in `testbench.yaml` under
`providers:` so Testbench boots it. Published files land in
`workbench/` by default, not `config/` or `resources/` of a host app.

## Source Layout

- `src/` — package source, PSR-4 autoloaded per `composer.json`
- `tests/` — Pest or PHPUnit suite, base case `Orchestra\Testbench\TestCase`
- `config/` — publishable defaults (the file shipped with the package)
- `resources/` — views, translations, Boost skills / guidelines
- `database/migrations`, `database/factories` — only if the package
  ships them
- `workbench/` — developer-only Testbench scaffolding; never shipped

Check sibling files before inventing structure. Do not introduce new
top-level directories without a clear reason.

## Cross-Version Compatibility

Packages usually support multiple Laravel / PHP majors simultaneously.

- Before using a feature, verify it exists in every version listed in
  `require` constraints.
- Prefer framework APIs present across the whole range over
  version-exclusive sugar.
- The CI matrix runs the suite against every supported combination —
  keep it green; do not drop a matrix leg to make a change pass.

## Conventions

- Match existing code style, naming, and structural patterns — check
  sibling files before writing new ones.
- Use descriptive names (`resolvePublishDestination`, not `resolve()`).
- Reuse existing helpers before adding new ones.
- Do not add dependencies without approval; every new `require` is a
  constraint downstream consumers inherit.

## Tests Are the Specification

The package has no running application to click through. Tests are how
behaviour is pinned down.

- Write tests alongside any behavioural change. Feature tests through
  Testbench are preferred over ad-hoc tinker scripts.
- Do not create "verification scripts" when a test can prove the same
  thing.
- Run `vendor/bin/pest` (or the project's equivalent) before claiming a
  change is done.

## Public API Discipline

- Every `public`, `protected`, or exported symbol is part of the
  package's surface. Breaking changes require a major version bump.
- Prefer `final` classes and `private`/`@internal` markers for anything
  not intended for extension.
- Keep config keys, published asset paths, and service container
  bindings stable across patch and minor versions.

## Documentation Files

Only create or edit documentation (README, CHANGELOG, docs/) when
explicitly requested or when a behaviour change requires it.

## Replies

Be concise. Focus on what changed and why. Skip restating what the
diff already shows.

---

# Release Automation

## CHANGELOG.md is updated automatically — do NOT edit by hand for releases

`CHANGELOG.md` is kept in sync with GitHub releases by `.github/workflows/update-changelog.yml`. When a release is published (not just drafted), the workflow uses `stefanzweifel/changelog-updater-action` to prepend the release body to `CHANGELOG.md` and commits the update back to `main`.

This means:

- **Do not** add changelog entries manually when preparing a release. The release body (drafted in `internal/release-notes-<version>.md` and pasted into the GitHub release) becomes the changelog entry automatically.
- **Do not** include a changelog diff in the release PR — the post-release commit comes from CI.
- If the changelog needs a fix *after* a release, edit `CHANGELOG.md` directly and commit — but this is unusual and only for typos or formatting issues in the auto-generated entry.

## Benchmark table in release body is updated automatically


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SanderMuller/laravel-fluent-validation](https://github.com/SanderMuller/laravel-fluent-validation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
