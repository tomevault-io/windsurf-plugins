---
trigger: always_on
description: - PHP 8.4+, Hyperf 3.2, Swoole 6.2 resident/coroutine server; `bin/hyperf.php` is the CLI entrypoint and `core.php` bootstraps configuration.
---

# LogShare — Agent Guide

## Stack and entrypoints

- PHP 8.4+, Hyperf 3.2, Swoole 6.2 resident/coroutine server; `bin/hyperf.php` is the CLI entrypoint and `core.php` bootstraps configuration.
- PSR-4 maps `App\` to `app/`; controllers use Hyperf annotation routes. `App\Controller\AbstractController` provides request parsing and response helpers.
- HTTP listens on `0.0.0.0:9501`; both deprecated `/1/` and current `/v1/` API routes are supported (a `/{version:v?1}` route prefix matches both). The `/rag` MCP endpoint is served by the same process and accepts loopback requests only unless `ai.mcp.rag.authToken` is configured.
- Storage is selected by `storage.storageId`: MariaDB (`s`) or filesystem (`f`). Redis is an optional cache/rate-limit dependency.
- `OpenLiteWaf/` and `OpenLiteStats/` are independent GitHub repos mounted as git submodules — they have their own READMEs, tests, and release lifecycle.

## Setup and verification

```bash
git submodule update --init --recursive   # OpenLiteWaf / OpenLiteStats 以 submodule 引入，clone 或 pull 部署后必须执行
composer install
cp Config.inc.example.php Config.inc.php
composer test
composer test:architecture
PHPSTAN_TURBO=0 composer stan  # Termux; omit the prefix on CI/Ubuntu
php bin/hyperf.php list
php bin/hyperf.php rag:build
php bin/hyperf.php start
```

- Tests use Pest and bootstrap through `tests/bootstrap.php`; that bootstrap creates `Config.inc.php` when absent and supplies a Redis mock when ext-redis is unavailable. Run one file with `vendor/bin/pest tests/Unit/FilterTest.php` or filter by name with `vendor/bin/pest --filter=...`; architecture tests are the `architecture` Pest group (`composer test:architecture`).
- Integration tests need MariaDB and Redis. CI initializes MariaDB with `docker/mariadb-init.sql`; local Docker services are started with `docker compose -f docker/compose.yaml up -d`. CI (`.github/workflows/ci.yaml`) also smoke-tests the booted server on `9501` (`/v1/limits`, `POST /v1/log`, `/rag` MCP JSON-RPC) and builds the Docker image.
- Lua regression tests (Termux): `php OpenLiteWaf/tests/openlitewaf_regex_test.php`, `lua5.1 OpenLiteWaf/tests/openlitewaf_logic_test.lua`, `lua5.1 OpenLiteStats/tests/openlitestats_logic_test.lua`, `luac5.1 -p OpenLiteWaf/lua/*.lua OpenLiteStats/lua/*.lua`. The two Lua test suites stub ngx and use real file IO under `/data/data/com.termux/files/usr/tmp/` for persistence cases.
- PHPStan analyzes `app/` at level 5. The two existing `ignoreErrors` entries in `phpstan.neon` are intentional (Codex type hierarchy in `app/Log.php`, Hyperf Response `getConnection()` in `app/Sse/SseWriter.php`), and `app/Client/SpinYarnClient.php` is excluded because it depends on the spinyarn PHP extension that is absent in the CI static-analysis environment; do not add suppressions or exclusions casually.
- There is no configured formatter. Before finishing code changes, run the relevant Pest tests, `composer test:architecture`, and `PHPSTAN_TURBO=0 composer stan` on Termux.

## Configuration and operational constraints

- Copy `Config.inc.example.php` to the gitignored `Config.inc.php`; never commit it or expose its API keys. Database and Redis connection settings can be overridden with `DB_*` and `REDIS_*`; AI settings in `.env` include `AI_ENABLED`, `AI_API_KEYS`, `AI_BASE_URL`, `AI_MODEL`, and JSON `AI_RAG_PROVIDERS`.
- Do not change `id.characters` or the ID length: existing log IDs depend on them. IDs are seven characters, with `s`/`f` identifying the storage backend.
- Upload limits are enforced before storage: 10 MB / 50,000 lines, plus at most 200 files and 12 MB total. ZIP uploads must remain protected against traversal and excessive expansion.
- SpinYarn is optional and only parses mappings already present under `mappings/`; it has no automatic download. The extension and mapping files are primarily handled by the Docker build/bind mount. `spinyarn_init()` gained a 5th `redis_url` parameter in SpinYarn v1.1.0 (the Dockerfile pin since 1.7.5); v1.0.0 accepted only 4 parameters — `SpinYarnClient::supportsRedisArg()` detects the loaded signature via reflection and adapts, because blindly passing 5 args to v1.0.0 throws ArgumentCountError and the fail-open path then disables deobfuscation for the whole process (2026-09 production incident). Since v1.1.0 the local LRU cache is gone (Redis-backed only; without a `redis_url` mappings are parsed on demand). Restart the server after swapping the extension .so.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NingZeStudio/LogShare](https://github.com/NingZeStudio/LogShare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
