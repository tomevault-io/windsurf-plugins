---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A standalone Laravel package (not an application) that signs PDF files with A1/x509 certificates, PKCS#12 or PEM, and cryptographically verifies existing PDF signatures. Published on Packagist as `lsnepomuceno/laravel-a1-pdf-sign`.

The invariants are imported rather than summarised, so they are in context for every session instead of being a link someone has to decide to follow:

@docs/spec/invariants.md

The v1 surface, the global helper functions plus `src/Sign/*` and `src/Entities/*`, is **gone**, not deprecated. `UPGRADE.md` maps every removed API to its replacement.

Documentation is split by lifecycle, and `tests/Project/SpecTest.php` fails when a reference into it stops resolving:

| Read | For |
|---|---|
| `docs/spec/invariants.md` | the rules that break the product or the project. **Read before touching `src/Signing`, `src/Validation` or the dependency list** |
| `docs/spec/public-api.md` | what the package exposes, and what changing it costs |
| `docs/spec/quality-policy.md` | the gates, and why each sits where it does |
| `docs/spec/conventions.md` | how the code is written. **Read before writing a helper or a class constant** |
| `docs/decisions/` | why the design is what it is: one numbered file per decision |
| `docs/history/v2-modernization.md` | why v1 was shaped as it was, and where the build diverged from the plan |
| `docs/history/decision-log.md` | which questions were put, and when they were answered |

`ARCHITECTURE.md` is the index. When you change behaviour that a decision record justifies, update that record's outcome section too: a record whose outcome is never written back is how the previous document drifted away from the code.

**A behaviour change is not finished until every surface that describes it says the same thing.** `CONTRIBUTING.md` enumerates them, and the list is enumerated rather than summarised because "and any other relevant documentation" is exactly what let three of them go stale at once: `samples/` sat a release behind, the documentation site stopped at 2.3.1 while 2.4 shipped, and the README never named two facade methods that had been public for a release. Three of those surfaces now have gates (`tests/Conformance/SamplesTest.php`, and two rules in `tests/Project/ArchTest.php` covering docblocks and the README's coverage of the facade); the rest are review.

**The documentation site lives on the `docs` branch, not here.** Nothing in a pull request against `main` can check it, and no test on `main` will ever fail because of it. It is updated in its own pull request, on the day a version is tagged, and it deliberately describes only what is installable: a feature merged and not yet tagged does not belong on it.

## Commands

```bash
composer check          # everything CI runs: pint --test, phpstan, deps, pest
composer test           # vendor/bin/pest
composer analyse        # PHPStan level max, no baseline
composer lint           # Pint (PER-CS); append --test to only check
composer deps           # unused/shadow dependency report
composer test:cov       # line coverage (needs pcov or xdebug)
composer test:types     # type coverage, gated at 100%
composer test:mutate    # mutation testing over Certificates, Signing and Validation

vendor/bin/pest tests/Signing/SigningTest.php                   # single file
vendor/bin/pest --filter="writes the CAdES sub-filter"   # single test
vendor/bin/pest --exclude-group=network                  # skip live TSA tests
vendor/bin/pest --parallel                               # 16 processes here; mutation needs it
```

Tests run on Orchestra Testbench, not a host app. `openssl` on `PATH` is **not** required to run the suite: `Testing\DebugCertificate` generates throwaway PKCS#12 bundles through the ext-openssl functions. The binary is only needed by `OpenSslCliCertificateReader` and `Validation\SignatureVerifier`.

Tests in the `network` group hit a live timestamp authority (freetsa.org) and fail offline.

Helpers shared across test files must live in `tests/Pest.php` (`debugCertificate()`,
`testCertificate()`, `resource()`). A helper defined inside one test file is invisible to the
others under `--parallel`, which fails as `Call to undefined function`.

A Husky `pre-commit` hook formats staged PHP files with Pint (`npm install` to enable it). It
falls back to the Docker service when the local PHP is older than Pint's 8.2 floor.

### Docker

The local floor is PHP 8.4 and the matrix reaches 8.5, so version-specific work goes through `.docker`:

```bash
docker compose -f .docker/compose.yaml run --rm php85 composer check
```

Services `php83` / `php` (8.4) / `php85`, each keeping `vendor/` in its own named volume. That volume **masks the host `vendor/`**, which is why PhpStorm reports missing classes after a Docker-only install. Fix it with `composer install --ignore-platform-reqs` on the host (documented in `CONTRIBUTING.md`).

CI (`.github/workflows/main_action.yml`) runs PHP 8.4 and 8.5 against Laravel 13, on pull
requests to `main` only. Keep it in sync with `composer.json` and the compatibility table in
`README.md`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lsnepomuceno/laravel-a1-pdf-sign](https://github.com/lsnepomuceno/laravel-a1-pdf-sign) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
