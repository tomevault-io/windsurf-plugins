---
trigger: always_on
description: > **SCOPE — read this first.** This directory **is the DuckPHP framework source**. Everything in this file is about **developing and maintaining the framework itself**: it is written for the framework's own maintainers and for coding agents that change `src/`, `tests/`, `docs/` or `demo/` **here**. It says **nothing** about how to build an application with DuckPHP.
---

# AGENTS.md — DuckPHP **framework development** (this repo) — NOT for apps that merely use DuckPHP

> **SCOPE — read this first.** This directory **is the DuckPHP framework source**. Everything in this file is about **developing and maintaining the framework itself**: it is written for the framework's own maintainers and for coding agents that change `src/`, `tests/`, `docs/` or `demo/` **here**. It says **nothing** about how to build an application with DuckPHP.
>
> **If you are here to *use* DuckPHP (not to change it), this is the wrong file — stop and take the proper route:**
> 1. in *your own* project: `composer require dvaknheo/duckphp`, then `./vendor/bin/duckphp new` to scaffold it (or copy `skeleton/` by hand);
> 2. the scaffolded project carries its **own** `AGENTS.md` (generated from [`skeleton/AGENTS.md`](skeleton/AGENTS.md)) — **that** is your project's convention sheet (layering, naming, directory tree, the four steps of adding a feature);
> 3. how to use the framework: [`docs/zh/guide/index.md`](docs/zh/guide/index.md) (52 chapters, English mirror under `docs/en/guide/`); per-class API and option defaults: [`docs/zh/reference/index.md`](docs/zh/reference/index.md). In an installed project they sit under `vendor/dvaknheo/duckphp/docs/`.
> 4. if you opened this file from `vendor/dvaknheo/duckphp/AGENTS.md` inside a user project: close it and read that project's own `AGENTS.md` instead. The rules below (framework-wide `src/` ASCII, 100% coverage gates, the Chinese/English docs trees, the `ZAllDemoTest` baselines, …) do **not** apply to a project that merely depends on DuckPHP.
>
> Audience for the rest of this file: AI agents / engineers working **in this repository** (the framework itself, not a user project). **Read this file first**; jump to the two maintenance guides when you need the long version.
> Language note: this file is English; the two maintenance guides and the documentation tree are Chinese (the project's working language) — that is intentional, not an oversight.

## 0. What this is

| Item | Value |
|---|---|
| Project | DuckPhp (formerly DNMVCS): zero-dependency, fully replaceable components, **library-style** PHP framework |
| Composer package | `dvaknheo/duckphp` |
| PHP | `>= 7.4.0` (**do not use 8.x syntax**; the measured runtime is PHP 8.2) |
| Framework source | `src/` (pure ASCII, see §3) |
| Tests | `tests/` (PHPUnit 9.6; `phpunit.xml` enables `processIsolation`) |
| Docs | `docs/zh/` (**source of truth**) + `docs/en/` (parallel tree, identical file names) |
| Demo / skeleton | `demo/` (living documentation *and* the E2E test host), `skeleton/` (the template that generates user projects) |
| CLI / containers | `bin/duckphp`, `docker/test-php74`, `docker/test-php84` |

## 1. Environment prerequisites (where things go wrong; verified 2026-10-02)

**Always run from WSL** (a Windows-side PHP has no redis extension, so the redis test cases fail spuriously). The repository is `E:\ProjectGoat\DNMVCS` on Windows and `/mnt/e/ProjectGoat/DNMVCS` in WSL; every command below runs from the repository root.

1. **`vendor/` must match `composer.json`.** This repository **does not commit `composer.lock`** (a library does not pin its dependencies), so the local dependencies come from `composer update`. Correct versions: `phpunit 9.6.37`, `php-code-coverage 9.2.32`, `dvaknheo/libcoverage 1.0.8`, `nikic/php-parser 5.x`.
   When versions are out of step you see these, and they mean:

   | Error | What is missing / wrong |
   |---|---|
   | `Call to undefined method LibCoverage\LibCoverage::_()` | libcoverage is too old (< 1.0.8) |
   | `Class "PHP_Token_AMPERSAND_FOLLOWED_BY_VAR_OR_VARARG" not found` | `php-token-stream` is too old; `covagg.php` and `tests/support.php` then read no dump at all |
   | `Class "PhpParser\ParserFactory" not found` | `nikic/php-parser` is missing (needed by `gen-architecture-gv.php`) |
   | `Class "Redis" not found` / `RedisException: Connection refused` | see the next point |

2. **The full test run also needs redis**: `apt-get install -y php8.2-redis redis-server`, then
   `redis-server --requirepass 123456 --save "" --appendonly no --daemonize yes`
   — the port and password match the `redis_list` in `tests/data_for_tests/setting.php` (the `docker/test-php74` compose file starts redis the same way). Without either one, `RedisCacheTest` / `RedisManagerTest` / `DuckPhpTest` / `Ext/RouteHookWebInstallerTest` turn red.

3. **Run the full suite as `XDEBUG_MODE=coverage php vendor/bin/phpunit`**: `phpunit.xml` uses process isolation, and `LibCoverage` inside the child processes needs xdebug's coverage mode; this machine's `xdebug.mode` defaults to `develop`, so **do not** run the full suite with `--no-coverage` (a batch of cases then fail in the children with `Code coverage needs to be enabled`).

4. `vendor/`, `composer.lock`, `test_reports/`, `test_coveragedumps/` and `backup/` are in `.gitignore` — **never commit them**.

## 2. Layout

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dvaknheo/duckphp](https://github.com/dvaknheo/duckphp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
