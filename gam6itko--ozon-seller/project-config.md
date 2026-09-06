---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`gam6itko/ozon-seller` — a PHP library (no framework) wrapping the Ozon Seller API (<https://docs.ozon.ru/api/seller>).
PSR-18 client / PSR-17 factories are injected by the consumer; the library ships no HTTP implementation of its own.

Supports **PHP 7.1 through 8.2** (CI matrix). Do not use syntax newer than 7.1 (no arrow functions, no typed properties,
no `??=`, no named arguments, no union types in signatures — use `@var`/psalm annotations instead).

## Commands

```shell
composer tests            # phpunit (all tests; they are fully mocked, no network)
vendor/bin/phpunit --filter testImport tests/Service/V1/ProductServiceTest.php
composer csfix            # php-cs-fixer fix (@Symfony + declare_strict_types + aligned =>)
composer psalm            # psalm --no-cache (errorLevel 4, src only)

docker compose up php71   # runs tests + psalm on the oldest supported PHP
docker compose up php80

php bin/is_realized.php              # diffs var/swagger.json against the library
php bin/is_realized.php path/to/spec.json
php bin/is_realized.php --download   # try to refresh the spec first (usually fails, see below)
php bin/is_realized.php | grep /v2/posting/fbs/get
```

`is_realized.php` works off a local spec file: the CLI argument, or `var/swagger.json`
(gitignored) by default. `--download` fetches `SWAGGER_URL` into that file first, but normally fails — docs.ozon.ru sits
behind a JS anti-bot challenge (`307 ...?__rr=1` → `403` +
`challengeURL`), so the spec has to be saved from a browser. A failed download never overwrites the existing file.
Progress messages go to stderr, so piping into `grep` still works.

`MAPPING` in that script only lists URLs whose method name can't be derived from the path — everything else is resolved
by `findMethod()`'s convention, so don't add entries the convention already covers. Paths Ozon removed from the spec are
dead weight there (the script iterates the spec, not `MAPPING`); the removed ones and their replacements are listed in a
comment at the end of the table.

php-cs-fixer 3.x as locked here refuses to run on PHP > 8.0 (this machine has 8.4); use
`docker compose up php80` or `PHP_CS_FIXER_IGNORE_ENV=1` — the latter at your own risk.

CI (`.github/workflows/`) runs PHPUnit on 7.1–8.2 and psalm on 7.4–8.1.

## Architecture

**Everything is a service class extending `Gam6itko\OzonSeller\Service\AbstractService`**, namespaced by Ozon API
version: `Service\V{1..6}\XxxService` and `Service\V{n}\Posting\{Fbs,Fbo,Crossborder}Service`. The version in the
namespace is the version in the URL path, not a library version — the same logical service exists at several versions in
parallel (e.g. `V1\ProductService` … `V5\ProductService`), and new Ozon endpoints go into a class matching their URL's
version segment.

`AbstractService` owns the whole transport concern:

- `parseConfig()` accepts either an assoc array (`clientId`, `apiKey`, `host`) or a positional list of up to 3 values;
  host defaults to `https://api-seller.ozon.ru`.
- `createRequest()` attaches `Client-Id` / `Api-Key` / `Content-Type` headers and json-encodes array bodies.
- `request()` unwraps the `result` key from the response by default (`$returnOnlyResult`) and converts errors via
  `throwOzonException()`.
- Error mapping is **convention-based, not a table**: the Ozon `error.code` string is snake_case → StudlyCase, a
  trailing `_ERROR` is dropped, and `Exception` is appended, producing
  `Gam6itko\OzonSeller\Exception\{Name}Exception`. Adding support for a new Ozon error code means adding a class with
  that derived name in `src/Exception/`; unknown codes fall back to `OzonSellerException`.

**Request payload shaping happens in the service methods**, using three helpers, and this is the pattern every method
follows:

- `Utils\ArrayHelper::pick($input, [...whitelist])` — silently drops keys Ozon doesn't accept.
- `TypeCaster::castArr($input, ['key' => 'int', ...])` — coerces scalar types.
- `ProductValidator` (`new ProductValidator('create'|'update', $version)`) — driven by
  `src/config/product_validator_v{1,2,3}.php`, which declares per-field `type`, `requiredCreate`,
  `requiredUpdate` and allowed `options`. Product field changes usually belong in those config files, not in the
  validator.
- `Utils\WithResolver` — normalizes the `with` flag block, whose allowed keys differ per (version, posting scheme,
  method); that matrix lives in `WithResolver::getKeys()`.
- `src/Enum/*` hold the API's string constants (`Visibility`, `Status`, `PostingScheme`, …).

Payload/response shapes are documented as `@psalm-type T*` blocks in the **class** docblock and referenced from method
`@param`/`@return` (see `V4\Posting\FbsService`, `V3\Posting\FbsService`). Derive them from `var/swagger.json`, but
describe the request as the method actually sends it — i.e. after `ArrayHelper::pick`, not the full spec schema — and
note where nested structures were left as bare `array`. A method implementing one of the marker interfaces must keep
`|array<array-key, mixed>` in its `@param`, otherwise psalm reports
`MoreSpecificImplementedParamType`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gam6itko/ozon-seller](https://github.com/gam6itko/ozon-seller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
