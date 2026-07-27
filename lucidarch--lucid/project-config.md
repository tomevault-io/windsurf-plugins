---
trigger: always_on
description: The `lucidarch/lucid` Composer package — the actual library that implements the Lucid Architecture pattern for Laravel. It ships a CLI binary (`lucid`), a set of base unit classes, a bus dispatcher, and code generators (stubs) for all Lucid units.
---

# lucidarch/lucid — Package CLAUDE.md

## What This Is

The `lucidarch/lucid` Composer package — the actual library that implements the Lucid Architecture pattern for Laravel. It ships a CLI binary (`lucid`), a set of base unit classes, a bus dispatcher, and code generators (stubs) for all Lucid units.

**Current version:** 2.0.0 (in progress — see upgrade plan)
**Packagist name:** `lucidarch/lucid`
**PHP:** ^8.1 | **Laravel:** 9.x, 10.x, 11.x, 12.x

---

## Repository Layout

```
lucid/
├── bin/
│   ├── test-local.sh        # Local integration test runner (see Testing section)
│   ├── test-commands.sh     # Integration smoke test — runs inside a Laravel project
│   └── test-packagist.sh    # Packagist-based install test
├── src/
│   ├── Bus/
│   │   ├── UnitDispatcher.php   # Core dispatch logic — run(), serve(), marshal()
│   │   └── ServesFeatures.php   # Trait mixed into controllers
│   ├── Units/
│   │   ├── Feature.php          # Base Feature class (uses UnitDispatcher)
│   │   ├── Job.php              # Base Job class
│   │   ├── Operation.php        # Base Operation class
│   │   ├── QueueableJob.php     # Queueable Job base
│   │   ├── QueueableOperation.php
│   │   ├── Controller.php       # Base controller (uses ServesFeatures)
│   │   └── Model.php            # Base model extending Eloquent
│   ├── Console/
│   │   ├── Command.php          # Base Symfony console command
│   │   └── Commands/            # All `lucid make:*`, `list:*`, `delete:*` commands
│   ├── Generators/
│   │   ├── stubs/               # PHP stub templates used by generators
│   │   ├── FeatureGenerator.php
│   │   ├── JobGenerator.php
│   │   ├── OperationGenerator.php
│   │   ├── ServiceGenerator.php
│   │   ├── MicroGenerator.php   # Generates the micro app structure
│   │   └── MonolithGenerator.php
│   ├── Providers/
│   │   └── RouteServiceProvider.php  # Base route provider (namespace-free, Laravel 9+)
│   ├── Entities/                # Value objects representing units in the codebase
│   ├── Testing/                 # MockMe, UnitMock, UnitMockRegistry
│   ├── Validation/              # Validation trait + validator
│   ├── Domains/Http/Jobs/       # Built-in HTTP response jobs
│   ├── Finder.php               # Locates units in the filesystem
│   ├── Filesystem.php
│   ├── Parser.php               # Parses unit names/paths
│   └── Str.php                  # String helpers
├── lucid                        # CLI entry point (Symfony Console Application)
├── composer.json
├── Dockerfile                   # php:8.2-alpine for CI
└── .github/workflows/tests.yml  # CI matrix
```

---

## Architecture Concepts (core mental model)

| Unit | Base Class | Location (Micro) | Location (Monolith) | Rule |
|------|------------|-------------------|----------------------|------|
| Job | `Lucid\Units\Job` | `app/Domains/{Domain}/Jobs/` | same | calls nothing |
| Operation | `Lucid\Units\Operation` | `app/Operations/` | `app/Services/{Svc}/Operations/` | calls Jobs only |
| Feature | `Lucid\Units\Feature` | `app/Features/` | `app/Services/{Svc}/Features/` | calls Jobs + Operations |
| Domain | — (directory) | `app/Domains/{Domain}/` | same | groups Jobs by topic |
| Service | — (directory) | N/A | `app/Services/{Svc}/` | groups Features/Operations by area |
| Model | `Lucid\Units\Model` | `app/Data/Models/` | same | Eloquent entity |

**Dispatch chain:** Controllers call `$this->serve(new SomeFeature(...))` → Features call `$this->run(new SomeJob(...))` or `$this->run(new SomeOperation(...))` → Operations call `$this->run(new SomeJob(...))`.

---

## Testing

### Unit tests (PHPUnit)
There are no PHPUnit tests in the package source itself — tests are integration tests run against a real Laravel install.

### Integration tests (primary test method)

**`bin/test-local.sh`** — run this for local development. Requires a sibling `../Testbench/` directory.

```sh
# Test all versions (9.x 10.x 11.x 12.x)
./bin/test-local.sh

# Test a single version
./bin/test-local.sh 11.x

# Test specific versions
./bin/test-local.sh 10.x 11.x

# Wipe and recreate installs before testing
FRESH=1 ./bin/test-local.sh

# Wipe a single version
FRESH=1 ./bin/test-local.sh 11.x
```

**What it does:**
1. Creates (or reuses) a Laravel project at `../Testbench/laravel-{VERSION}/`
2. Registers `lucid/` as a Composer path repository (symlinked — changes are live immediately)
3. Copies `bin/test-commands.sh` into the Laravel root and runs it

**`bin/test-commands.sh`** — runs inside a Laravel project. Tests every CLI command end-to-end:
- Generates controllers, features, jobs, models, operations, policies for both Micro and Monolith
- Calls `php -l` on every generated file
- Runs `./vendor/bin/phpunit`
- Tears down all generated files

**Testbench directory location:** `../Testbench/` relative to this repo (i.e., sibling of `lucid/` inside `lucidarch/`). Must exist before running `test-local.sh`.

### CI (GitHub Actions)

Matrix: PHP 8.1/8.2/8.3/8.4 × Laravel 9.x/10.x/11.x/12.x × prefer-lowest/prefer-stable.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lucidarch/lucid](https://github.com/lucidarch/lucid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
