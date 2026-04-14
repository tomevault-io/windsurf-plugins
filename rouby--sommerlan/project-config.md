---
trigger: always_on
description: The **Sommerlan App** is a full-stack TypeScript web application for managing summer LAN party events. It handles attendee registration, room allocation, payment tracking, donations (via PayPal), purchase/snack shop, event scheduling, photo galleries, and Discord/email integrations. The live app is deployed at [sommerlan.rocks](https://sommerlan.rocks) and [staging.sommerlan.rocks](https://staging.sommerlan.rocks).
---

# Copilot Instructions for Sommerlan App

## Project Overview

The **Sommerlan App** is a full-stack TypeScript web application for managing summer LAN party events. It handles attendee registration, room allocation, payment tracking, donations (via PayPal), purchase/snack shop, event scheduling, photo galleries, and Discord/email integrations. The live app is deployed at [sommerlan.rocks](https://sommerlan.rocks) and [staging.sommerlan.rocks](https://staging.sommerlan.rocks).

---

## Repository Structure

This is a **Yarn 4 monorepo** with three workspaces:

```
sommerlan/
├── .github/                     # CI/CD workflows, issue templates
├── docs/                        # Architecture docs (e.g. auth flow)
├── helm/                        # Kubernetes Helm chart for deployment
├── packages/
│   ├── server/                  # Node.js/Fastify/GraphQL backend
│   ├── client/                  # React/TypeScript frontend
│   └── integration/             # Playwright E2E tests
├── Dockerfile                   # Multi-stage production build
├── docker-compose.yml           # Local Docker environment
├── graphql.config.ts            # GraphQL tooling configuration
└── package.json                 # Root workspace + scripts
```

### Server (`packages/server/src/`)
```
index.ts          # Entry point (loads dotenv, newrelic)
server.ts         # Fastify + GraphQL Yoga setup
config.ts         # Server configuration (port, dev mode)
auth.ts           # WebAuthn & JWT authentication
ability.ts        # CASL-based authorization (RBAC)
schema/           # GraphQL schema modules (one folder per domain)
  party/          # Party management (queries, mutations, resolvers)
  user/           # User profile & authentication
  donation/       # PayPal donations
  game/           # Party games
  purchase/       # Snack shop purchases
  money/          # Money transfers
  admin/          # Admin operations
  events/         # Event scheduling
  overwatch/      # Overwatch game features
data/             # Google Sheets ORM layer
  $base.ts        # Abstract base class for all data models
  $cache.ts       # In-memory cache with JSON Patch sync
  *.ts            # Individual model files (User, Party, Attending, etc.)
services/         # External service integrations
  mail.ts         # Nodemailer email sending
  discord.ts      # Discord.js bot integration
  paypal.ts       # PayPal SDK integration
  scheduler.ts    # Cron job scheduler (recron)
  webAuthn.ts     # WebAuthn passkey helpers
```

### Client (`packages/client/src/`)
```
main.tsx              # React entry point
App.tsx               # Root component (Mantine provider, router)
GraphQLProvider.tsx    # URQL GraphQL client configuration
routes/               # TanStack Router pages (index, admin, imprint)
features/             # Feature modules (PartyCheckIn, Purchases, etc.)
components/           # Shared UI components (NavLink, UserAvatar, etc.)
hooks/                # Custom React hooks
state/                # Jotai atoms for client state
gql/                  # Auto-generated GraphQL types & hooks
```

### Integration Tests (`packages/integration/`)
```
tests/                # Playwright test files
playwright.config.ts  # Playwright configuration (base URL: http://127.0.0.1:5173)
```

---

## Tech Stack

| Layer        | Technology |
|-------------|-----------|
| **Backend** | Node.js 20+, TypeScript, Fastify, GraphQL Yoga |
| **Frontend** | React 18, TypeScript, Vite, TanStack Router, URQL, Mantine UI |
| **Database** | Google Sheets (via `google-spreadsheet` library) |
| **Testing**  | Vitest (unit), Playwright (E2E/integration) |
| **Auth**     | WebAuthn (passkeys), JWT tokens, CASL (authorization) |
| **Build**    | SWC (server), Vite (client) |
| **Deploy**   | Docker, Kubernetes, Helm, GitHub Actions |
| **Monitoring**| New Relic APM |
| **Package Manager** | Yarn 4.4.0 (PnP mode) |

---

## Getting Started

### Prerequisites
- Node.js ≥ 20
- Yarn 4.4.0 (`corepack enable` and `corepack prepare yarn@4.4.0 --activate`)

### Install Dependencies
```bash
yarn
```
> **Note:** Yarn 4 uses PnP (Plug'n'Play) in loose mode. Do NOT use `npm` or `pnpm` — they will not work with the PnP resolution.

### Run Development Server
```bash
yarn dev
```
This concurrently starts:
- **Backend** on port **2022** (or `PORT` env var) with nodemon + pino-pretty logging
- **Frontend** on port **5173** (Vite dev server)

Access the app at **http://localhost:5173**. The frontend proxies GraphQL requests to the backend.

### Environment Variables (Development)
Create a `.env` file in `packages/server/` or set these variables:
```bash
NODE_ENV=development
SESSION_SECRET=<any-random-string>
APP_ORIGIN=http://localhost:5173
# For Google Sheets data access (required for full functionality):
GOOGLE_SHEET_ID=<sheet-id>
GOOGLE_SERVICE_ACCOUNT_EMAIL=<email>
GOOGLE_PRIVATE_KEY=<key>
# Optional integrations:
DISCORD_BOT_TOKEN=<token>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rouby) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
