---
trigger: always_on
description: === foundation rules ===
---

<bagisto-guidelines>
=== foundation rules ===

# Bagisto Guidelines

Bagisto is a Laravel-based e-commerce platform. These guidelines are specifically curated for developing with Bagisto and its package-based architecture.

## Foundational Context

This application is a **Bagisto** e-commerce platform built on Laravel 12. You must be familiar with both Laravel and Bagisto's modular package architecture.

### Technology Stack

- **PHP**: 8.3+
- **Laravel**: v12
- **Vue.js**: For admin panel interactivity
- **Tailwind CSS**: For styling
- **Laravel Octane**: v2
- **Laravel Sanctum**: v4
- **Laravel Socialite**: v5
- **Laravel Boost**: v2
- **Laravel MCP**: v0
- **Laravel Pint**: v1
- **Pest**: v3
- **PHPUnit**: v11

### Bagisto Core Packages

Bagisto uses a modular package structure in `packages/Webkul/`:

| Package | Purpose |
|---------|---------|
| **Admin** | Admin panel functionality |
| **Shop** | Customer storefront |
| **Core** | Common utilities and helpers |
| **Product** | Product management |
| **Category** | Category management |
| **Checkout** | Cart and checkout process |
| **Payment** | Payment methods (CashOnDelivery, MoneyTransfer) |
| **Paypal** | PayPal integration |
| **Shipping** | Shipping methods |
| **Sales** | Order management |
| **Customer** | Customer management |
| **Attribute** | Product attributes |
| **Inventory** | Stock management |
| **CartRule** | Cart promotions |
| **CatalogRule** | Catalog promotions |
| **DataGrid** | Admin data tables |
| **Tax** | Tax calculation |
| **CMS** | Content management |
| **Theme** | Theme management |

## Skills Activation

This project has domain-specific skills available. You MUST activate the relevant skill whenever you work in that domain—don't wait until you're stuck.

- `package-development` — Package development in Bagisto. Activates when creating packages, migrations, models, repositories, routes, controllers, views, localization, DataGrid, menus, ACL, or system configuration. Use references: @core (package structure, service providers), @data (migrations, models, repositories), @ui (routes, controllers, views), @features (localization, DataGrid, menus, ACL, system config).

- `shipping-method-development` — Shipping method development in Bagisto. Activates when creating shipping methods, integrating shipping carriers like FedEx, UPS, DHL, or any third-party shipping provider; or when the user mentions shipping, shipping method, shipping carrier, delivery, or needs to add a new shipping option to checkout.

- `payment-method-development` — Payment gateway development in Bagisto. Activates when creating payment methods, integrating payment gateways like Stripe, PayPal, or any third-party payment processor; or when the user mentions payment, payment gateway, payment method, Stripe, PayPal, or needs to add a new payment option to the checkout.

- `product-type-development` — Product type development in Bagisto. Activates when creating custom product types, defining product behaviors, or implementing specialized product logic. Use references: @config (product type configuration), @abstract (AbstractType methods), @build (complete subscription implementation).

- `shop-theme-development` — Shop theme development in Bagisto. Activates when creating custom storefront themes, modifying shop layouts, building theme packages, or working with Vite-powered assets for the customer-facing side of the application.

- `admin-theme-development` — Admin theme development in Bagisto. Activates when creating custom admin themes, modifying admin layouts, building admin theme packages, or working with admin panel styling and interface customization.

- `pest-testing` — Tests applications using the Pest 3 PHP framework. Activates when writing tests, creating unit or feature tests, adding assertions, testing Livewire components, architecture testing, debugging test failures, working with datasets or mocking; or when the user mentions test, spec, TDD, expects, assertion, coverage, or needs to verify functionality works.

- `blade-conventions` — Blade template conventions for any Bagisto package (Admin, Shop, or a custom Webkul-style module). Activates when creating or editing Blade views, building anonymous `@props` or Vue-backed `x-template` components, wiring forms, datagrids, modals, layouts, or slots; or when matching the project's markup, attribute-binding, indentation, comment, and formatting style.

- `bagisto-api-develop` — Install / remove / extend the `bagisto-api` package (REST + GraphQL). Activates when installing or removing the package, or adding/changing an endpoint, resource, or admin menu's API; or when the user mentions `ApiResource`, `Provider`, `Processor`, `DTO`, "install the package", or "add an endpoint". Install/remove run only on explicit request.

- `bagisto-api-shop` — Build a storefront app/UI on the Shop API (`/api/shop/*` + `/api/graphql`). Activates when building a customer-facing storefront, catalog/cart/checkout flow, customer account, or shopping app; or when the user mentions products, cart, checkout, coupons, wishlist, or customer login/account.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bagisto/agent-skills](https://github.com/bagisto/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
