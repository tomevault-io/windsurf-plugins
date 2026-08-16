---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

This is the original source code of the `swoole/library` package — the PHP-land portion of the [Swoole](https://www.swoole.com) extension. The code here is compiled into the C header file `ext-src/php_swoole_library.h` of [swoole-src](https://github.com/swoole/swoole-src) (via swoole-src's `tools/build-library.php`) and shipped inside the extension itself. The version number of this package always matches the Swoole extension version.

Because the library also ships embedded in the extension, tests and scripts must run PHP with `-d swoole.enable_library=Off` so the source files in this repository are loaded instead of the copy built into the installed extension. The `composer test` script already does this.

## Swoole / PHP Version Support Matrix

| Swoole series | Supported PHP versions      |
|---------------|-----------------------------|
| 6.2           | 8.2, 8.3, 8.4, 8.5          |
| 6.1           | 8.1, 8.2, 8.3, 8.4          |
| 6.0           | 8.1, 8.2, 8.3, 8.4          |
| 5.1           | 8.0, 8.1, 8.2, 8.3          |
| 5.0           | 8.0, 8.1, 8.2               |
| 4.8           | 7.2, 7.3, 7.4, 8.0, 8.1, 8.2 |

Since Swoole 6.0, ZTS (Zend Thread Safety) builds are available; the `Swoole\Thread` classes and their tests require a ZTS build.

## Common Commands

Development happens inside Docker (the `app` container runs `phpswoole/swoole` with all backing services linked: MySQL, PostgreSQL, Oracle, Redis, MongoDB, Consul, Nacos, PHP-FPM, WordPress).

```bash
# Start the environment
docker compose up -d

# Install dependencies inside the app container (so composer.lock matches the
# container's PHP version; composer.lock is git-ignored)
docker compose exec app composer update -n

# Run the full test suite
docker compose exec app composer test

# Run a single test file
docker compose exec app php -d swoole.enable_library=Off ./vendor/bin/phpunit tests/unit/StringObjectTest.php

# Run a single test by name
docker compose exec app php -d swoole.enable_library=Off ./vendor/bin/phpunit --filter=testSplit tests/unit/StringObjectTest.php

# Coding style (PHP-CS-Fixer; CI runs the same script with -- --dry-run)
docker compose exec app composer cs-fix

# Static analysis (PHPStan level 5 over ./src)
docker compose exec app ./vendor/bin/phpstan analyse --no-progress --memory-limit 2G

# Run examples
docker compose exec app php examples/mysqli/base.php
```

Notes:

- Many tests depend on the Docker services defined in `docker-compose.yml`; connection constants (hosts, credentials) are defined in `tests/bootstrap.php`. Give services (especially Oracle/MySQL) time to boot before running database tests.
- The HTTP/curl tests (`tests/unit/Coroutine/HttpFunctionTest.php`, most of `tests/unit/Curl/HandlerTest.php`) query the `httpbin` service of `docker-compose.yml` (`mccutchen/go-httpbin`, a local stand-in for httpbin.org), which the app container reaches as `local.httpbin.org` on port 80 — no external network access is involved. Mind one difference when writing assertions: go-httpbin reports request values (args, headers, form) as arrays of strings, where httpbin.org reports single values as plain strings.
- Tooling that loads the Composer autoloader (`composer cs-fix`, PHPStan, PHPUnit) fails on any PHP whose Swoole extension has the embedded library enabled ("Constant SWOOLE_LIBRARY already defined", "Cannot redeclare function"). To run the test suite *against* the embedded library, `composer-embedded.json` installs the test tooling (PHPUnit, mongodb/mongodb, the test-helper classmap) without the library's own autoload rules: `COMPOSER=composer-embedded.json composer update`, then `php ./vendor/bin/phpunit`. The `app` container sets `swoole.enable_library=off` in its php.ini, so these commands work there. On a host with Swoole installed, `php -d swoole.enable_library=Off vendor/bin/php-cs-fixer fix` works for the fixer, but PHPStan needs the setting in an ini file because its worker processes do not inherit `-d` flags.
- PHPStan resolves Swoole symbols from the `swoole/ide-helper` stubs, wired up through `scanDirectories` in `phpstan.neon.dist`, so the analysis gives the same result on NTS and ZTS builds. The stubs are what supply `SWOOLE_THREAD` and the `Swoole\Thread` classes, which an NTS build does not expose. Because the stubs are more complete than the extension's own reflection, several `@phpstan-ignore` comments are unnecessary and must not be reintroduced — an unmatched ignore is itself a non-ignorable error. Note that `swoole/ide-helper` is required as `dev-master`, so a stub change can start or stop an error at any time.
- `tests/unit/Thread` is excluded from the default PHPUnit suite because it requires a ZTS build of PHP/Swoole.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swoole/library](https://github.com/swoole/library) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
