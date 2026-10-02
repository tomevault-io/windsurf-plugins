---
trigger: always_on
description: This file follows the [agents.md](https://agents.md) cross-tool standard. It's a portable summary of how to be effective on a Spree Commerce codebase, written for any agentic CLI (Codex, Cursor, Copilot, Aider, Windsurf, Zed, Amp, etc.) that reads `AGENTS.md`.
---

# Spree Commerce — Agent Guidance

This file follows the [agents.md](https://agents.md) cross-tool standard. It's a portable summary of how to be effective on a Spree Commerce codebase, written for any agentic CLI (Codex, Cursor, Copilot, Aider, Windsurf, Zed, Amp, etc.) that reads `AGENTS.md`.

**Targets Spree 6.x (Rails 8.1).** For Spree 5.x projects, use the `v0.3.0` tag of this repository.

If you're running in **Claude Code**, install this package as a plugin and you'll get the 38 SKILL.md files under `skills/` as on-demand context, the `spree-expert` subagent, two slash commands, and two safety hooks. See [README.md](./README.md) for install instructions.

If you're running in **any other tool**: read this file, then dive into the relevant `skills/<name>/SKILL.md` when the task matches its domain.

---

## What Spree is

Spree Commerce is an open-source, self-hosted, API-first commerce platform built on Ruby on Rails. The thing people choose it for is the ability to customize and extend it without forking. Architecture:

1. **Backend (Ruby gems)** — `spree_core` (models, services, workflows), `spree_api` (Store, Admin and Seller REST APIs under `/api/v3/`), `spree_dashboard` (serves the admin dashboard at `/dashboard`), `spree_emails` (transactional email), plus provider gems (`spree_stripe`, `spree_meilisearch`, `spree_easypost`, …).
2. **TypeScript SDKs** — `@spree/sdk` (Store API), `@spree/admin-sdk` (Admin API), `@spree/seller-sdk` (Seller API).
3. **Admin UI** — `@spree/dashboard`, a React app you own and extend with plugins (`apps/dashboard/`). An optional seller dashboard exists for marketplaces.
4. **Storefront** — the reference Next.js storefront (`apps/storefront/`), or anything you build on the Store API.

Users run Spree in their own infrastructure. Everything is opt-in customization.

## Core conventions (don't violate these without a reason)

### Ruby / Rails

- All Spree code is namespaced under `Spree::`; models inherit from `Spree.base_class`, not `ApplicationRecord` directly.
- Use `Spree.customer_class` (default `Spree::Customer`) and `Spree.admin_user_class` (default `Spree::AdminUser`) — never hardcode the class. `Spree.user_class` is deprecated.
- Always scope queries through the current store (`current_store.products`, not `Spree::Product.all`). Multi-store apps share a database; un-scoped queries leak data across stores. `Spree::Store.default` can return `nil` — set `Spree::Current.store` in jobs, rake tasks and tests.
- **Cart and Order are separate models.** `Spree::Cart` (`cart_…`) is the shopping/checkout phase; completing it creates an immutable `Spree::Order` (`or_…`). Records owned by either side (line items, payments, fulfillments, tax lines, discounts, fees) use `#owner`.
- **No state machines.** Status fields are plain string `status` columns declared with `has_status`; transitions happen in `Spree::Workflow` classes. Don't add state machines and don't convert statuses to Rails enums.
- **Side effects go in workflow hooks or event subscribers**, not `after_*` callbacks or decorators on business methods. `Spree.hooks.register('carts.complete.before_finalize', …)` for in-flow logic; `publish_event` + subscribers for reactions.
- Money adjustments are typed rows: `Spree::TaxLine`, `Spree::Discount`, `Spree::Fee`. Prices are per currency: `variant.price_in(currency)` / `variant.set_price(currency, amount)`.
- IDs are strings at the API surface (Stripe-style prefixed IDs); never `.to_i` an ID.
- `belongs_to` is required by default — declare `optional: true` where blank is legitimate.
- Uniqueness validations use `scope: spree_base_uniqueness_scope` plus a DB index. Always pass `class_name` and `dependent` on associations.
- Permit extra attributes on the model: `Spree::Product.additional_permitted_attributes += [:brand_id]` (never `<<`).
- Merchant-managed data belongs in **custom fields**; schemaless private data in `metadata`.

### API v3 (REST)

Three surfaces under `/api/v3/`:

- **Store API** (`/api/v3/store/*`) — customer-facing. Auth: publishable key (`pk_*`) + optional customer JWT. Customer access is ownership-scoped.
- **Admin API** (`/api/v3/admin/*`) — back-office. Auth: secret key (`sk_*`, scoped `read_*`/`write_*`) or staff JWT. Both pass the same per-endpoint permission check; staff permissions come from roles stored as data.
- **Seller API** (`/api/v3/seller/*`) — marketplace sellers. Seller JWT + `X-Spree-Seller-Id`.

All share prefixed IDs (`prod_…`, `cart_…`, `or_…`, `variant_…`), `{ data, meta }` list envelopes, Ransack filters (`q[name_cont]=...`), `expand=...` and `fields=...`, and money as strings. See `skills/spree-api-v3/SKILL.md`.

### TypeScript

- Use `@spree/sdk` / `@spree/admin-sdk` / `@spree/seller-sdk` to call the API from TypeScript.
- For custom endpoints, use the SDK's `client.request<T>(method, path, options)` escape hatch or extend the client with a wrapped resource class — don't fork the SDK and don't bypass it with raw `fetch`.

## Project layout and commands


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spree/agent-skills](https://github.com/spree/agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
