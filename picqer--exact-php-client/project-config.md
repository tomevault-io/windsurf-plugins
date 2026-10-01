---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

`picqer/exact-php-client` is a PHP library (PHP >= 7.4, Guzzle 7; see `composer.json` for the minimum version) that acts as an API client for the Exact Online REST API. It is open source and widely used, so avoid breaking changes to public behavior. Entity classes mirror Exact's own naming and conventions, so the Exact API reference is the source of truth for endpoints and field names.

- Resource overview: https://start.exactonline.nl/docs/HlpRestAPIResources.aspx?SourceAction=10
- Resource detail pages: `https://start.exactonline.nl/docs/HlpRestAPIResourcesDetails.aspx?name={Service}{Resource}` (for example, `LogisticsItems` or `SyncLogisticsItems`). Each entity's `@see` tag links to its page.

## Commands

```bash
composer install
vendor/bin/phpunit                                   # full test suite
vendor/bin/phpunit tests/ModelTest.php               # single file
vendor/bin/phpunit --filter testCanFindModel         # single test
vendor/bin/phpstan                                   # static analysis (level 5, uses phpstan-baseline.neon)
vendor/bin/phpstan --generate-baseline phpstan-baseline.neon   # regenerate baseline
```

CI runs phpunit on PHP 7.4 through 8.5 on pushes and pull requests, plus a `--prefer-lowest` run on PHP 7.4 and 8.5 so the minimum dependency versions are tested. phpstan runs on PHP 7.4 for pull requests. The `PHP x.y` jobs and `Static analysis` are required status checks on `main`, so don't rename them without updating the branch protection. Keep code compatible with PHP 7.4: no enums, readonly properties, constructor promotion, union types or `never`/`mixed` return types (use a `@return never` docblock instead). StyleCI applies the `recommended` preset plus `concat_with_spaces` and `not_operator_with_successor_space` (`! $foo`).

## Releases

This library is widely used, so follow semver strictly:
- **Patch:** only fixes that users won't notice except that something broken now works.
- **Minor:** anything that changes observable behaviour (different exceptions, errors that used to be swallowed) or raises a dependency minimum. List these under "Behaviour changes" in the release notes.

Releases are GitHub releases with a `vX.Y.Z` tag on `main`. Packagist picks them up automatically. `CHANGELOG.md` is historical and no longer maintained; the release notes are the changelog.

## Architecture

All code lives in the `Picqer\Financials\Exact` namespace under `src/Picqer/Financials/Exact/`.

- **`Connection`** handles OAuth2 (authorization code, access and refresh tokens, and lock/unlock/update callbacks around token refresh), request building through Guzzle, response parsing (JSON `d`/`results` unwrapping, `__next` pagination via `nextUrl`, and XML for the XML upload/download endpoints), division handling, file downloads (`downloadFile()`), and rate-limit headers (`waitOnMinutelyRateLimitHit`).
  - URLs are built as `{apiUrl}/{division}/{endpoint}`. A `{division}` placeholder in the endpoint is filled in instead (beta endpoints). Absolute `http(s)://` URLs are used as is. Only `SystemUser` and `Me` skip the division (see `requiresDivisionInRequestUrl`).
  - Every error becomes an `ApiException`. For HTTP errors the code is the status code; for connection errors it is `0`.
  - `Connection::$nextUrl` is shared state that every request overwrites. Code that paginates must copy it right after its own request (see `Findable::collectionFromResultAsGenerator`) and must never read it later.
- **`Model`** is the abstract base class for all entities. It stores attributes, filters them through `$fillable`, and serializes to JSON. Magic `__get` lazy-loads `__deferred` navigation properties by mapping the property name to a class: the trailing `s` is stripped (for example, `SalesInvoiceLines` maps to `SalesInvoiceLine`), so those entity class names must follow that convention. If the load fails, the `ApiException` is thrown. If no matching class exists, the raw `__deferred` array is returned. Array values assigned via `__set` go into `$deferred` and are only sent on insert (`json(0, true)`).
- **Traits** add capabilities to each entity:
  - `Query\Findable` provides `find`, `findWithSelect`, `findId`, `filter`, `first`, `get`, `getResultSet`, the `*AsGenerator` variants, and OData `$filter`/`$expand`/`$select` support. `Query\Resultset` handles paginated fetching.
  - `Persistance\Storable` provides `save` (insert or update based on `exists()`), `insert`, `update`, and `delete`. Updates and deletes address records as `{url}(guid'{primaryKey}')`.
  - `Persistance\Downloadable` downloads binary content from an entity's `getDownloadUrl()` through `Connection::downloadFile()`.
  - `Webhook\Authenticatable` verifies incoming webhook signatures.

### Entities

There are about 470 entity files. Each file is a thin class containing:
- A docblock with `@see` pointing to the Exact API docs page and one `@property` per field. PHPStan relies on these properties.
- `protected $fillable = [...]`, which lists every field name exactly as Exact spells it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [picqer/exact-php-client](https://github.com/picqer/exact-php-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
