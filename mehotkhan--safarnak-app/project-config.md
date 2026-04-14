---
trigger: always_on
description: Safarnak App – Cursor AI Repository Rules
---

Safarnak App – Cursor AI Repository Rules

Project Overview
- Single-root monorepo for a full-stack, offline-first travel app.
- Client: Expo React Native (Android/iOS/Web) with NativeWind v4.
- Server: Cloudflare Worker (GraphQL Yoga), Cloudflare D1 via Drizzle ORM.
- Shared: GraphQL schema and operations under `graphql/` used by both client and worker.
- Shared: Database schema (`database/schema.ts`) shared between client and worker - unified schema with UUID IDs, separate adapters for server (D1) and client (expo-sqlite).
- Database: `database/` folder contains unified schema with separate adapters - same table definitions, different database backends.

Current Stack (as of v1.13.0)
- TypeScript ~5.9.3
- React 19.1.0, React Native 0.81.5, Expo ~54.0.20
- Expo Router ~6.0.13, NativeWind ^4.1.21, Tailwind ^3.4.17
- Apollo Client 3.8.0, GraphQL ^16.11.0, GraphQL Yoga ^5.16.0
- Drizzle ORM ^0.44.7 (Shared schema for both server & client), Cloudflare D1 (server), Expo SQLite (client)
- Subscriptions: `graphql-workers-subscriptions` ^0.1.6 with Durable Objects
- ESLint ^9.38.0, Prettier ^3.6.2

Repository Structure
```
worker/                 # Cloudflare Worker resolvers and entry
graphql/                # Shared GraphQL schema + operations
api/                    # Client GraphQL types & hooks (generated, with automatic Drizzle sync)
database/               # Shared database schema and adapters
├── schema.ts         # Unified schema with UUIDs (server + client tables in one file)
├── server.ts         # Server adapter (Cloudflare D1) - exports getServerDB()
├── client.ts         # Client adapter (Expo SQLite) - exports getLocalDB(), sync utilities
├── index.ts          # Main exports (re-exports from schema, server, client)
├── types.ts          # Database types
└── utils.ts          # UUID utilities (createId, isValidId)
migrations/           # Server-only migrations (Cloudflare D1, at project root)
store/                  # Redux Toolkit store, slices, middleware
app/                    # Expo Router pages
components/             # UI components & contexts
constants/, hooks/, locales/ ...
```

Worker
- Entry point: `worker/index.ts` (defined in `wrangler.toml` main).
- Uses `readGraphQLSchema` from `graphql/schema-loader.ts`.
- Resolvers under `worker/queries`, `worker/mutations`, `worker/subscriptions`.
- Subscriptions enabled with Durable Object `SubscriptionPool`.

GraphQL & Codegen
- Schema: `graphql/schema.graphql`.
- Operations: `graphql/queries/*.graphql`.
- Generate client code: `yarn codegen` → `api/types.ts`, `api/hooks.ts`.
- Never edit `api/types.ts` or `api/hooks.ts` manually.

Path Aliases (TypeScript & Metro)
- Use ONLY path aliases; avoid all relative imports.
```
"@/*"           → "./*"
"@components/*" → "./components/*"
"@graphql/*"    → "./graphql/*"
"@database/*"   → "./database/*"   # All database schemas & client utilities
"@worker/*"     → "./worker/*"
"@api"          → "./api"
"@api/*"        → "./api/*"
"@store"        → "./store"
"@store/*"      → "./store/*"
"@hooks"        → "./hooks"
"@hooks/*"      → "./hooks/*"
"@constants"    → "./constants"
"@constants/*"  → "./constants/*"
```

Updated aliases (tsconfig + Metro):
```
"@/*"           → "./*"              # e.g., import from '@/store/...'
"@ui/*"         → "./ui/*"
"@graphql/*"    → "./graphql/*"
"@database/*"   → "./database/*"
"@worker/*"     → "./worker/*"
"@api"          → "./api"
"@api/*"        → "./api/*"
"@state"        → "./ui/state"       # UI local state utilities
"@state/*"      → "./ui/state/*"
"@hooks"        → "./ui/hooks"
"@hooks/*"      → "./ui/hooks/*"
"@constants"    → "./constants"
"@constants/*"  → "./constants/*"
"@locales"      → "./locales"
"@locales/*"    → "./locales/*"
"@assets"       → "./assets"
"@assets/*"     → "./assets/*"
```

GraphQL Endpoint Configuration (Client)
- Preferred: set `expo.extra.graphqlUrl` via `app.config.js` (reads from env vars).
- Environment variables (in priority order):
  - `EXPO_PUBLIC_GRAPHQL_URL_DEV` or `EXPO_PUBLIC_GRAPHQL_URL` (Expo inlines these at build time)
  - `GRAPHQL_URL_DEV` or `GRAPHQL_URL` (runtime env vars)
- Dev fallback: `http://192.168.1.51:8787/graphql` (or auto-derived from Metro host)
- Production default: `https://safarnak.app/graphql`
- Sources: `api/client.ts` (URI resolution), `app.config.js` (configures `expo.extra.graphqlUrl`)

Critical Rules
1) Never add `"type": "module"` to `package.json`.
2) Use `eslint.config.mjs`; do not convert ESLint config type.
3) Keep strict separation:
   - `graphql/` is shared between client and worker.
   - `api/` is client-only, generated hooks with automatic Drizzle sync.
   - `worker/` is server-only code.
   - `database/schema.ts` is SHARED between client and worker - single source of truth:
     * `schema.ts` - Unified schema defining both server tables (users, trips, etc.) and client cached tables (cachedUsers, cachedTrips, etc.) with UUID IDs
     * `server.ts` - Server adapter: `getServerDB(d1)` for Cloudflare D1 (worker resolvers use this)
     * `client.ts` - Client adapter: `getLocalDB()` for Expo SQLite (client components use this)
     * Both adapters import from the same `schema.ts` file, ensuring schema consistency
     * `index.ts` - Exports all schemas and utilities

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mehotkhan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
