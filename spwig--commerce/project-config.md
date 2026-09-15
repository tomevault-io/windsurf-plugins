---
trigger: always_on
description: handles it.
---

# CLAUDE.md

Notes for Claude Code (and any other AI coding assistant) working on this
repo. Human contributors should read [CONTRIBUTING.md](CONTRIBUTING.md) and
[ARCHITECTURE.md](ARCHITECTURE.md) first — this file assumes you've
skimmed both.

Every rule below is a non-negotiable constraint. Read the section that
matches your change before you write code.

---

## ⚠️ This file is PUBLIC — read before editing it

`CLAUDE.md` is tracked in git and ships in the public AGPL repository.
Anything you add here is published to everyone, permanently, and cannot be
recalled by a later delete — it stays in history.

It is deliberately tracked (see `.gitignore`) so that contributors working
in a fresh clone get the same design invariants the maintainers use. That
only works if it stays free of anything install-specific.

**Never add to this file:**

- Absolute filesystem paths (`/mnt/...`, `/home/...`, `C:\...`). Use
  repo-relative paths.
- Hostnames, IP addresses, ports, or fleet/server topology.
- Credentials, tokens, keys, or anything resembling one.
- Links to paths that are gitignored — `.claude/`, `.claude_code/`,
  `internal_tools/`, `docs/`. A link a public clone cannot follow is worse
  than no link, and describing tooling that does not ship misleads
  contributors into trying to use it.
- Customer, merchant, or employee names; support tickets; revenue figures.
- Anything you would not put in a public README.

**Before you edit it, ask:** is this rule true for anyone who clones this
repo, or only true on this machine? Only the first kind belongs here.
Machine-specific notes belong in your own ignored config.

`make check-claude-md` (also run by pre-commit and CI) enforces the
mechanical half of this. It cannot judge the rest — that is on you.

---

## What Spwig is

Self-hosted e-commerce platform. Django 5, PostgreSQL, Redis, Docker.
AGPL-3.0-or-later. Every solution should be built with ease of use,
modern aesthetics, and speed, with the merchant and their customers at
the centre of focus.

**Branding rule.** Spwig merchants and their customers don't know about
Django. When you write documentation, error messages, admin copy, or any
merchant/customer-facing text, use "Spwig" instead of naming the
underlying framework.

## Terminology

- **Merchant** — the store owner/admin who runs a Spwig install.
- **Community / Pro / Enterprise** — runtime editions gated by a signed
  licence file. Community is the default when the platform
  auto-bootstraps a licence.
- **Provider** — a pluggable component that integrates an external
  system (payments, shipping, exchange rates, translation, GeoIP, SMS,
  email). Lives in [Spwig/components](https://github.com/Spwig/components).
- **Marketplace / update server** — `updates.spwig.com`; where
  components are published and where anonymous telemetry pings go.
  Client code lives in `component_updates/`.

---

## Architecture: single-tenant by design

Each Spwig installation is **one store, one merchant, one Django Site**.

- `SITE_ID = 1` is hard-coded. There is no multi-site UI.
- Django's Sites framework exists because `django-allauth` needs it —
  not because we use it for multi-tenancy.
- The Sites admin is unregistered in `core/admin.py` so merchants
  can't create additional sites.
- All code assumes `Site.objects.get(pk=1)`.
- Multi-store merchants run one Spwig install per store.
- **Do NOT implement multi-site / multi-tenant features.**

Models with Site foreign keys (`EmailAccount`, `EmailTemplate`,
`EmailOutbox`, `ExchangeRateProviderAccount`) always use Site ID=1.

More background: [ARCHITECTURE.md](ARCHITECTURE.md).

---

## Editions & licence bootstrap

- `core.apps:CoreConfig.ready()` bootstraps a signed **Community**
  licence at `settings.LICENSE_PATH` on first boot. Template lives at
  `core/data/community_licence.json`.
- `ActivationMiddleware` becomes a no-op once any valid licence file
  exists — Community installs never see an `/activate/` redirect loop.
- Feature-gating is **runtime**, not build-time.
  `LicenseManager.is_community()` /
  `is_hosted_service_available('geoip')` are the gates.
  Don't strip Pro-only code from the OSS build — keep it and gate.
- **POS module.** `pos_app/` and `pos_api/` ship in every edition
  including Community. `pos_app/license.py` returns True for every
  install as a no-op shim — Spwig no longer monetises POS as a
  feature. Revenue comes from hosting, hosted-service tier limits,
  the mail gateway, and support.

### When something is Pro-only

Two gating patterns:

1. **`LicenseManager.is_community()`** — early-returns a "requires paid
   tier" response.
2. **`is_hosted_service_available('service_name')`** — Community can
   use `geoip` / `geocoder` / `push` (rate-limited by the hosted
   service); only `mail_gateway` is Pro-only.

Prefer #2 when adding features that consume Spwig-hosted infrastructure.

---

## URL structure and routing

Decide before touching URLs: does the endpoint belong **inside**
`i18n_patterns` (localised URLs, `/en/...`) or **outside** (APIs and
webhooks, no language prefix)?

### Hierarchy

- **Non-i18n group** (no language prefix) lives outside `i18n_patterns`
  in `core/urls.py`. Use for APIs (`/api/...`), webhooks
  (`/webhooks/...`), health endpoints, `/i18n/`, and dev static/media
  helpers.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Spwig/commerce](https://github.com/Spwig/commerce) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
