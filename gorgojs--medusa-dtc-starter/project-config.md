---
trigger: always_on
description: A production-ready Medusa starter for direct-to-consumer commerce, published by Gorgo as a fork of
---

# AGENTS.md

## Overview

A production-ready Medusa starter for direct-to-consumer commerce, published by Gorgo as a fork of
the official [medusajs/dtc-starter](https://github.com/medusajs/dtc-starter). The repository is a
pnpm workspace holding a Medusa 2 backend (`@dtc/gorgo-medusa-backend`) and an optional Next.js 15
storefront (`@dtc/gorgo-medusa-storefront`).

People install this repository as a template through `create-medusa-app --repo-url`, so treat every
file as something a shop owner will read and then edit. The demo catalog, the placeholder copy and
the `supersecret` defaults are all deliberate.

## Where the Documentation Lives

The READMEs are the reference for installation, environment variables, commands and deployment. Read
them before answering a question about any of those, and update them when you change what they
describe.

- [README.md](README.md) covers the repository, its features and the full getting-started walkthrough
- [apps/backend/README.md](apps/backend/README.md) covers backend commands and every backend variable
- [apps/storefront/README.md](apps/storefront/README.md) covers storefront commands and every
  storefront variable, when `apps/storefront/` exists

Long-form guides live at
[docs.gorgojs.com/tools/medusa-dtc-starter](https://docs.gorgojs.com/tools/medusa-dtc-starter) and
are written in the `gorgojs/medusa-integrations` repository, not here.

## Directory Structure

```text
.
├── apps/
│   ├── backend/                      # Medusa application, Admin and transactional emails
│   │   ├── medusa-config.ts          # modules, plugins, feature flags, all env-driven
│   │   ├── eslint.config.mjs         # read by medusa lint, develop and build
│   │   ├── jest.config.js            # suites split by TEST_TYPE
│   │   ├── integration-tests/        # setup.js, referenced by jest.config.js setupFiles
│   │   ├── scripts/                  # copy-migration-data.js, runs after medusa build
│   │   └── src/
│   │       ├── admin/                # scaffold for Admin extensions and their i18n
│   │       ├── api/                  # custom store and admin routes, file-based
│   │       ├── emails/               # React Email templates, their i18n (36 locales) and lib/
│   │       │                         # lib/styles.ts is the one style sheet for all of them
│   │       ├── jobs/                 # scheduled jobs
│   │       ├── links/                # module links
│   │       ├── migration-scripts/    # initial-data-seed.ts, its stages under lib/, JSON under data/
│   │       ├── modules/              # smtp-notification provider
│   │       ├── subscribers/          # transactional emails and storefront revalidation
│   │       └── workflows/            # workflows and steps
│   └── storefront/                   # OPTIONAL Next.js 15 storefront
│       ├── messages/                 # 36 next-intl UI catalogs
│       ├── next.config.js            # standalone output, next-intl plugin, image hosts
│       ├── tailwind.config.js        # Medusa UI preset plus a hand-kept content allowlist
│       ├── public/flags/             # vendored country flags
│       └── src/
│           ├── app/                  # [locale] routes, api/, llms.txt, sitemap, robots
│           ├── i18n/                 # locale list, routing, request config, navigation
│           ├── lib/                  # data loaders, constants, hooks, utils, geolocation
│           ├── middleware.ts         # locale and region resolution
│           ├── modules/              # feature areas: store, products, cart, checkout, account…
│           └── styles/globals.css    # Tailwind entry and the few global overrides
├── pnpm-workspace.yaml
├── turbo.json
└── package.json
```

**`apps/storefront` may not exist.** It ships alongside the backend in a storefront-facing project,
but a backend-only project — one that demonstrates only backend integrations or a plugin a shop's
storefront never touches — omits it. Before running any storefront command, referencing storefront
files, or assuming a full-stack change is possible, check that `apps/storefront/` exists. If it
doesn't, the project is backend-only — do not scaffold it or suggest it was deleted by mistake.

The Medusa convention directories (`admin`, `api`, `jobs`, `links`, `modules`, `subscribers`,
`workflows`) each keep a `README.md` from the framework describing the primitive they hold. Read the
local one before adding a file there. `emails` and `migration-scripts` are this starter's own and
carry no such README.

Those seven names belong to Medusa's loaders, and `subscribers` is walked recursively. Every `.ts`
file under it is imported at boot and validated as a subscriber, so a helper parked anywhere inside
warns on every start. Code that is not one of the seven primitives belongs in a sibling directory of
its own, the way `emails` and `migration-scripts` already do.

## Package Manager

pnpm 10.11.1, pinned by `packageManager` in the root [package.json](package.json). Node 20.19 or
later, or 22.12 or later. `engines` excludes v21.

When the pnpm on `PATH` is a different major, run `corepack pnpm <command>`. A mismatched major asks
to delete and reinstall every `node_modules` in the workspace.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gorgojs/medusa-dtc-starter](https://github.com/gorgojs/medusa-dtc-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
