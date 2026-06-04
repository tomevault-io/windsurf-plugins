---
trigger: always_on
description: > Deenruv — Open-source headless commerce framework built with TypeScript, NestJS, and GraphQL (forked from Vendure).
---

# Repository Guidelines

> Deenruv — Open-source headless commerce framework built with TypeScript, NestJS, and GraphQL (forked from Vendure).

## Project Structure & Module Organization

```
deenruv/
├── apps/
│   ├── server/              # NestJS GraphQL API server (admin-api + shop-api)
│   ├── panel/               # React/Vite admin UI (Tailwind CSS, Zustand, React Router)
│   └── docs/                # Documentation site (Next.js + Fumadocs)
├── packages/
│   ├── core/                # Core framework (NestJS modules, entities, services)
│   ├── common/              # Shared types, utilities, generated GraphQL types
│   ├── admin-dashboard/     # Admin panel shell (layout, routing, providers)
│   ├── react-ui-devkit/     # UI SDK for building admin plugin UIs
│   ├── admin-types/         # Shared GraphQL/TypeScript types for admin
│   ├── admin-ui/            # Legacy Angular admin UI
│   ├── admin-ui-plugin/     # Plugin to serve legacy admin UI
│   ├── cli/                 # CLI tools
│   ├── create/              # Project scaffolding (`create-deenruv`)
│   ├── testing/             # E2E test utilities and helpers
│   ├── inpost/              # InPost API client library
│   ├── ui-devkit/           # Legacy Angular UI devkit
│   └── ts-node-register/    # TS-Node registration helper
├── plugins/                 # Official plugins (32+), naming: <feature>-plugin
├── e2e-common/              # Shared Vitest E2E config, test data, reporters
├── scripts/                 # Codegen, docs generation, import checks
│   ├── codegen/             # GraphQL type generation
│   ├── docs/                # TypeScript & GraphQL docs generation
│   └── changelogs/          # Changelog generation
└── docs/                    # Additional documentation files
```

### Naming Conventions

- **Plugins**: Always `<feature>-plugin` (e.g., `payments-plugin`, `seo-plugin`, `cronjobs-plugin`).
- **Packages**: Use the `@deenruv/` scope (e.g., `@deenruv/core`, `@deenruv/react-ui-devkit`).
- **Package names in `package.json`** may differ from directory names; always check.

### Plugin Architecture

Each plugin can extend both the server and the React admin UI:

```
plugins/<feature>-plugin/
├── src/
│   ├── plugin-server/       # Server-side logic
│   │   ├── index.ts         # Plugin definition (DeenruvPlugin)
│   │   ├── types.ts         # Configuration types
│   │   ├── services/        # NestJS services
│   │   ├── controllers/     # REST controllers
│   │   ├── handlers/        # Payment/shipping handlers
│   │   └── extensions/      # GraphQL schema extensions
│   └── plugin-ui/           # Admin UI extensions
│       ├── index.ts         # UI plugin (createDeenruvUIPlugin)
│       ├── components/      # React components
│       └── locales/         # i18n translations (JSON)
├── e2e/                     # E2E tests (*.e2e-spec.ts)
├── package.json
└── README.md
```

### Official Plugins (32)

| Plugin | Description |
|--------|-------------|
| `payments-plugin` | Mollie & Stripe payment integrations |
| `email-plugin` | Event-driven email system |
| `asset-server-plugin` | Local file serving + image transforms (S3/MinIO) |
| `elasticsearch-plugin` | Elasticsearch product search |
| `job-queue-plugin` | Background job processing (BullMQ/PubSub) |
| `przelewy24-plugin` | Przelewy24 + BLIK payments |
| `reviews-plugin` | Product reviews system |
| `seo-plugin` | SEO metadata management |
| `dashboard-widgets-plugin` | Admin dashboard widgets |
| `inpost-plugin` | InPost shipping integration |
| `cronjobs-plugin` | Scheduled cron jobs |
| `harden-plugin` | Security hardening |
| `merchant-plugin` | Multi-merchant product feeds (Google/Facebook) |
| `newsletter-plugin` | Newsletter subscriptions |
| `sentry-plugin` | Sentry error tracking |
| `stellate-plugin` | Stellate CDN/caching |
| `replicate-plugin` | Replicate AI integration |
| `replicate-simple-bg-plugin` | Simple background removal via Replicate |
| `redis-strategy-plugin` | Redis-based cache/session strategy |
| `wfirma-plugin` | wFirma invoicing integration |
| `upsell-plugin` | Upsell/cross-sell product suggestions |
| `order-reminder-plugin` | Order reminder notifications |
| `order-attributes-plugin` | Custom order attributes |
| `product-badges-plugin` | Product badge labels |
| `product-options-fields-plugin` | Extended product option fields |
| `method-modal-plugin` | Payment/shipping method selection modals |
| `facet-harmonica-plugin` | Facet filtering UI (harmonica style) |
| `copy-order-plugin` | Duplicate/copy existing orders |
| `in-realization-plugin` | Order realization tracking |
| `phone-number-validation` | Phone number validation |
| `apollo-cache` | Apollo server caching utilities |
| `deenruv-examples-plugin` | Example/demo plugin for development |

## Build, Test, and Development Commands

### Prerequisites

- **Node.js** >= 18
- **pnpm** (workspace-enabled, version 10.7.0+)
- **Docker** & Docker Compose

### First-Time Setup

```bash
pnpm install                  # Install all dependencies
pnpm server-docker-up         # Start Postgres, Redis, MinIO
pnpm build                    # Build all packages (sequential, respects deps)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aexol-studio/deenruv](https://github.com/aexol-studio/deenruv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
