---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Payum is a framework-agnostic PHP payment processing library. Every payment operation goes through a
**Gateway** that routes **Request** objects to **Action** objects, which may interrupt the flow by throwing
**Reply** exceptions (redirects, forms, raw responses).

- **Monorepo**: `src/Payum/Core` plus one directory per gateway package. Each package has its own
  `composer.json` and is published as a read-only split repo (`Payum/Core`, `Payum/Stripe`, …).
- **Gateways in this repo**: AuthorizeNet/Aim, Be2Bill, Klarna (Checkout, Invoice), Offline, Payex,
  Paypal (ExpressCheckout, ProCheckout, ProHosted, Masspay, Ipn, Rest), Sofort, Stripe (Checkout, Js,
  Direct), plus `Skeleton/` as the template for new ones. The "50+ gateways" figure comes from the Omnipay
  bridge (`payum/omnipay-v3-bridge`) and third-party packages, not from this repo alone.
- **PHP 8.1+**; default branch is `2.x` (renamed from `master`).


### Testing

```bash
# Full suite (~3200 tests). Both vars matter. Without SYMFONY_DEPRECATIONS_HELPER the run exits 1 on
# unsilenced deprecation notices from third-party SDKs even though every test passes (CI sets it
# globally). Without XDEBUG_MODE=off, MiddlewarePipelineTest::testShouldStopAHandlerThatDispatches-
# ItsWayIntoALoop fails outright — Xdebug's 512-frame nesting limit throws before Payum's own
# EndlessCycleDetector can. That failure is an artefact, not a regression.
XDEBUG_MODE=off SYMFONY_DEPRECATIONS_HELPER=weak vendor/bin/phpunit

# One component / one class / one method
vendor/bin/phpunit src/Payum/Core/Tests
vendor/bin/phpunit src/Payum/Core/Tests/GatewayTest.php
vendor/bin/phpunit --filter testMethodName src/Payum/Core/Tests/GatewayTest.php
```

`phpunit.xml.dist` sets `failOnRisky` and `stopOnRisky`, so a risky test aborts the run.

### Code quality (all three are CI gates)

```bash
vendor/bin/phpstan                     # level 6, reads phpstan-baseline.neon
vendor/bin/ecs check                   # coding standards
vendor/bin/ecs check --fix             # autofix
vendor/bin/rector process --dry-run    # CI runs this report-only
vendor/bin/rector process              # apply
```

CI runs PHPStan with `doctrine/mongodb-odm` installed (`composer require doctrine/mongodb-odm
--with-all-dependencies`); locally, without it, the ODM-related results differ.

## Architecture

### Execution flow

`Gateway::execute()` (`src/Payum/Core/Gateway.php`) is the whole runtime in ~50 lines:

1. Push a `Context` onto a stack (nested `execute()` calls make this a stack — that is what
   `EndlessCycleDetectorExtension` inspects).
2. `extensions->onPreExecute()`, then `findActionSupported()` picks the first action whose
   `supports($request)` returns true, then `extensions->onExecute()`.
3. `$action->execute($request)`, then `onPostExecute()`.
4. A thrown `ReplyInterface` is caught, passed to `onPostExecute()` (extensions may swap or clear it), then
   re-thrown — unless `execute($request, true)` was used, which returns the reply instead.
5. Other exceptions go through `onPostExecuteWithException()`, which preserves the exception chain the way
   Symfony's `ExceptionListener` does.

Actions delegate by creating sub-requests: implement `GatewayAwareInterface`, use `GatewayAwareTrait`, then
`$this->gateway->execute(new ObtainCreditCard(...))`.

### Two generations of gateway wiring (important)

This is the main thing in flight on `2.x`. Both mechanisms are live at once.

**Legacy (v1) config-array style — still what every shipped gateway factory uses:**

- `{Name}GatewayFactory extends GatewayFactory` and overrides `populateConfig(ArrayObject $config)`.
- `GatewayFactory::createConfig()` layers config: caller's config → factory defaults → `CoreGatewayFactory`
  config.
- `CoreGatewayFactory::create()` builds a PHP-DI container out of that flat array, then binds services to
  the gateway **by key prefix**: `payum.action.*` → `addAction()`, `payum.extension.*` → `addExtension()`,
  `payum.api.*` → `addApi()` (deprecated). Ordering is controlled by `payum.prepend_actions` /
  `payum.prepend_extensions` or by implementing `PrependActionInterface` / `PrependExtensionInterface`.
- `create()`, `createConfig()`, `buildActions()`, `buildApis()`, `buildExtensions()` and `buildClosures()`
  all emit deprecations now. Do not build new features on them.

**Current (v2) container style:**

- `Payum\Core\DI\ContainerConfiguration` is the target interface: `configureContainer(): array` returns
  PHP-DI definitions, `createGateway(ContainerInterface): Gateway` assembles the gateway.
- Only `CoreGatewayFactory` implements it so far; `GatewayFactory` carries a comment saying it will
  implement it (and drop `GatewayFactoryInterface`) in 3.0. `PayumBuilder` emits a deprecation for every
  factory that still lacks it.
- `PayumBuilder::getPayum()` builds a **global container** with the services shared by all gateways (token
  storage, `TokenFactoryInterface`, `GenericTokenFactoryInterface`, `HttpRequestVerifierInterface`, PSR-18
  client and PSR-17 factories, storage extensions). Then per gateway it layers a container:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Payum/Payum](https://github.com/Payum/Payum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
