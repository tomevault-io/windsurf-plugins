---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ChamberOrchestra View Bundle is a Symfony 8.0 bundle that provides a typed, reusable view layer for building JSON responses. Controllers return `ViewInterface` objects instead of `Response`; the bundle's event subscriber handles serialization to `JsonResponse` automatically.

**Requirements**: PHP 8.5+, Symfony 8.0 components, doctrine/common ^3.5

## Commands

```bash
composer install                        # Install dependencies
vendor/bin/phpunit                      # Run all tests (unit + integration)
vendor/bin/phpunit --filter ClassName   # Run a specific test class
vendor/bin/phpunit --filter testMethodName # Run a specific test method
composer test                           # Alias for vendor/bin/phpunit
composer analyse                        # Run PHPStan static analysis
composer cs-check                       # Check code style (dry-run)
composer cs-fix                         # Fix code style
composer bench                          # Run PHPBench benchmarks
php -l path/to/File.php                 # Quick syntax check
```

## Architecture

### View Hierarchy

The core abstraction is `ViewInterface` (marker interface). `ResponseViewInterface` defines `getStatus()` and `getHeaders()` for views that control HTTP response details. Views compose into JSON responses:

- **View** — abstract base class implementing `ViewInterface`; convenience superclass for custom views
- **ResponseView** — base with status code (200) and JSON headers; implements `ResponseViewInterface` and `NormalizableInterface`
- **DataView** — wraps any `ViewInterface` or array under a `"data"` key; implements `ResponseViewInterface`
- **BindView** — extends `stdClass`; maps matching properties from a source domain object using reflection via `BindUtils::sync()`. Uses a static `setBindUtils()`/`getBindUtils()` bridge to receive the DI-managed `BindUtils` instance (falls back to `new BindUtils()` without DI). The `#[BindsFrom(EntityClass::class)]` attribute declares source classes for targeted cache warming. The `#[Type(ViewClass::class)]` attribute on `IterableView` properties specifies element view classes
- **IterableView** — maps collections via callback or view class string
- **KeyValueView** — produces associative array output for metadata blocks
- **SourceCacheSignatureInterface** — extends `ViewInterface`; declares `static createCacheSignature(object $source): string` so signatures are computable from the source without constructing the view. Required by `CachedView`, `CachedBindView` and `#[Type(..., cached: true)]`
- **CachedViewInterface** — extends `CacheableViewInterface` with `getTtl()` and `createView()`; the contract `ViewNormalizer` resolves through the per-signature normalized-payload cache. Implemented by `CachedView` and `PrivateCachedView`
- **CachedView** — final descriptor pairing a source object with the view class rendering it (`new CachedView($user, UserView::class, ?factory, ?ttl)`); default factory is `new $viewClass($source)`. Carries NO status/headers (always the standard `DataView` envelope, use `ResponseView` for custom responses)
- **PrivateCachedView** — private-payload `CachedView` (composition over an inner `CachedView`): the current user's identifier and the request locale — resolved via `SecurityBridge`/`LocalisationBridge`, never passed in — are appended to the signature (`...@user:<identifier|anonymous>@locale:<locale|default>`), isolating cache entries per user × locale. For custom scoping, views use the aware-traits inside their own `createCacheSignature()` with a plain `CachedView`
- **SecurityAwareTrait** — gives views static access to the current user (`self::getUser()` / `self::getUserIdentifier()`), usable in constructors for personalised fields and in `createCacheSignature()`. Delegates to `SecurityBridge` (`src/Security/`), a static holder filled per request by `SetSecuritySubscriber` — same pattern as `BindView::setBindUtils()`. Trait statics are per-using-class in PHP, hence the single shared holder. symfony/security-core is a soft dependency (dev-only + composer suggest); without it everything resolves to the anonymous user. Worker-mode safe (FrankenPHP/RoadRunner/Swoole): the bridge stores the token storage SERVICE (reset between requests by the framework's services resetter), never a resolved user — do not change it to cache the user object statically
- **LocalisationAwareTrait / LocalisationBridge / SetLocalisationSubscriber** — the identical bridge pattern for the request locale: the bridge (`src/Localisation/`) holds the framework `RequestStack` service and resolves `getLocale()` from the current request at call time. No extra dependencies (http-foundation is already required). Same worker-mode rule: store the stack service, never a resolved locale
- **CacheableViewInterface** — extends `ViewInterface`; views implementing `getCacheSignature()` get their serialized JSON cached automatically by `ViewSubscriber` (the view is still constructed; only normalization + encoding are skipped). `CachedView` implements it

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chamber-orchestra/view-bundle](https://github.com/chamber-orchestra/view-bundle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
