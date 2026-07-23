---
trigger: always_on
description: Shop at Home is a full-stack shopping list application that lets customers securely add, edit, view, and remove grocery and household items. The project is designed as a learning resource for Azure Static Web Apps and Azure Container Apps, providing **four frontend implementations** (Angular, React, Svelte, Vue) backed by **two API options** (Azure Functions and Fastify).
---

# Shop at Home — Contributor & Agent Guide

## Project Overview

Shop at Home is a full-stack shopping list application that lets customers securely add, edit, view, and remove grocery and household items. The project is designed as a learning resource for Azure Static Web Apps and Azure Container Apps, providing **four frontend implementations** (Angular, React, Svelte, Vue) backed by **two API options** (Azure Functions and Fastify).

Live demos: [angular.shopathome.dev](https://angular.shopathome.dev) · [react.shopathome.dev](https://react.shopathome.dev) · [svelte.shopathome.dev](https://svelte.shopathome.dev) · [vue.shopathome.dev](https://vue.shopathome.dev)

---

## Repository Structure

```
shopathome/
├── angular-app/          # Angular 18 frontend (NgRx state management)
│   ├── src/app/          # Components, services, routing, store
│   └── package.json
├── react-app/            # React 17 frontend (Redux + Redux-Saga)
│   ├── src/              # Components, store, product views
│   └── package.json
├── svelte-app/           # Svelte 4 frontend (Vite build, TypeScript)
│   ├── src/              # Components, store, models
│   └── package.json
├── vue-app/              # Vue 3 frontend (Vuex state management)
│   ├── src/              # Components, views, store, router
│   └── package.json
├── api/                  # Azure Functions API (serverless)
│   ├── products-get/     # GET /api/products
│   ├── products-post/    # POST /api/products
│   ├── products-put/     # PUT /api/products
│   ├── products-delete/  # DELETE /api/products
│   ├── discounts-get/    # GET /api/discounts
│   └── shared/           # Shared data layer (in-memory)
├── fastify-api-server/   # Fastify 4 API (containerized)
│   └── src/
│       ├── server.js     # Server entry point (port 3000)
│       ├── routes/       # Route handlers (products, discounts)
│       └── shared/       # Shared data layer (in-memory)
├── .devcontainer/        # Dev container config (Svelte + API)
├── .github/workflows/    # Azure Static Web Apps deploy workflows
├── CONTRIBUTING.md       # Contribution guidelines
└── README.md             # Project overview and deployment guide
```

---

## Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| **Angular frontend** | Angular, NgRx, TypeScript, Karma/Jasmine | Angular 18, TS ~5.4 |
| **React frontend** | React, Redux, Redux-Saga, JavaScript | React 17 |
| **Svelte frontend** | Svelte, TypeScript, Vite | Svelte 4, Vite 4 |
| **Vue frontend** | Vue 3, Vuex, Vue Router, JavaScript | Vue 3 |
| **Azure Functions API** | Azure Functions (Node.js) | v4 runtime |
| **Fastify API** | Fastify, CORS, Helmet | Fastify 4 |
| **CSS** | Bulma, SCSS, Font Awesome | Bulma 0.9 |
| **Runtime** | Node.js | ≥ 20 |
| **Package manager** | npm | (lockfiles present per app) |
| **Deployment** | Azure Static Web Apps, Azure Container Apps | — |
| **Dev tools** | Prettier, ESLint, TSLint (Angular), SWA CLI | — |

---

## Build & Run

Each frontend app and each API is an independent npm project. Install and run them separately.

### Prerequisites

- Node.js ≥ 20
- [Azure Functions Core Tools](https://docs.microsoft.com/azure/azure-functions/functions-run-local) (for the Azure Functions API)
- [SWA CLI](https://www.npmjs.com/package/@azure/static-web-apps-cli) (for local integration)

### Install dependencies (per app)

```bash
cd angular-app && npm install
cd react-app && npm install
cd svelte-app && npm install
cd vue-app && npm install
cd api && npm install
cd fastify-api-server && npm install
```

### Run a frontend with the Azure Functions API

```bash
# Example: Svelte + Azure Functions
cd svelte-app
npm run start-svelte-func-swa
```

### Run a frontend with the Fastify API

```bash
# Example: Svelte + Fastify
cd svelte-app
npm run start-svelte-fastify
```

### Build a frontend

```bash
cd angular-app && npm run build     # ng build --configuration production
cd react-app && npm run build       # react-scripts build
cd svelte-app && npm run build      # vite build
cd vue-app && npm run build         # vue-cli-service build
```

### Run the Fastify API standalone

```bash
cd fastify-api-server && npm start  # Starts on http://localhost:3000
```

### Run the Azure Functions API standalone

```bash
cd api && npm start                 # func start (port 7071)
```

---

## Testing

| App | Test Runner | Command |
|-----|-------------|---------|
| angular-app | Karma + Jasmine | `cd angular-app && npm test` |
| react-app | Jest (via react-scripts) | `cd react-app && npm test` |
| svelte-app | No test setup | — |
| vue-app | No test setup | — |
| api | No test setup | — |
| fastify-api-server | No test setup | — |

When adding new features, add tests to the Angular and React apps at minimum. Follow existing patterns:

- **Angular**: Test files use `*.spec.ts` alongside components in `src/app/`.
- **React**: Test files use `*.test.js` alongside components in `src/`.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johnpapa/shopathome](https://github.com/johnpapa/shopathome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
