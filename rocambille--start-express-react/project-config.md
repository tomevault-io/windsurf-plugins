---
trigger: always_on
description: > AI agent briefing for this codebase. Read this before writing or modifying any code.
---

# AGENTS.md — StartER (start-express-react)

> AI agent briefing for this codebase. Read this before writing or modifying any code.
> Human-readable docs live in the [wiki](https://github.com/rocambille/start-express-react/wiki).

---

## Stack

- **Backend**: Node.js + Express 5, TypeScript, Zod (validation), `node:sqlite` (sync API)
- **Frontend**: React 19, React Router (with SSR/hydration), Vite, Pico CSS
- **Database**: SQLite — zero-config, synchronous, file at `data/sqlite/database.sqlite`
- **Tooling**: Biome (lint + format), Vitest (tests), tsx (runtime), Docker (optional)

---

## Directory structure

```
.
├── server.ts                  # Single entry point — bridges Express + Vite
├── index.html                 # Vite root — contains <!--ssr-outlet-->
├── data/
│   ├── mailpit/               # Mailpit persistence (Docker)
│   └── sqlite/
│       └── database.sqlite    # Generated locally — NOT committed to git
├── src/
│   ├── entry-client.tsx       # Client-side hydration (hydrateRoot)
│   ├── entry-server.tsx       # SSR rendering (renderToPipeableStream)
│   ├── database/
│   │   ├── schema.sql         # SQLite schema — source of truth for DB structure
│   │   └── seeder.sql         # Test/seed data
│   ├── express/
│   │   ├── routes.ts          # Registers all Express modules via importAndUse()
│   │   ├── helpers/           # Infrastructure: cache, validation, converters
│   │   └── modules/           # Business modules (item/, user/, auth/, ...)
│   │       └── <name>/
│   │           ├── <name>Routes.ts            # Route declarations
│   │           ├── <name>Actions.ts           # Request handlers (thin, delegate to repo)
│   │           ├── <name>ParamConverter.ts    # Converts URL params to entity-typed objects
│   │           ├── <name>Repository.ts        # All SQL queries for this entity
│   │           └── <name>Validator.ts         # Zod schema + validate middleware
│   ├── react/
│   │   ├── routes.tsx         # React Router route tree
│   │   ├── helpers/           # Hooks, mutations, fetch utilities
│   │   └── components/        # UI components and pages
│   └── types/
│       └── index.d.ts         # Shared TypeScript types (Item, User, etc.)
├── tests/
│   └── contracts              # API contract definitions — declarative source of truth
└── biome.json                 # Lint + format config
```

---

## Common commands

### Development

```bash
npm install
cp .env.sample .env
npm run database:sync      # Load schema + seed data into SQLite
npm run dev                # Start dev server (Express + Vite together on port 5173)
```

### Database

```bash
npm run database:schema:load          # Apply schema.sql to the SQLite file
npm run database:seeder:load          # Load seeder.sql test data
npm run database:sync                 # Both above — resets DB to a clean state
npm run database:sync -- -n           # Non-interactive (CI/CD — skips confirmation prompt)
npm run database:schema:load -- -n    # Same, schema only
npm run database:seeder:load -- -n    # Same, seeder only
```

> SQLite requires NO Docker, NO connection string, NO async setup. The DB file is created on the fly.

### Code quality (run before every commit)

```bash
npm run types:check     # TypeScript strict check (tsc --noEmit)
npm run biome:check     # Lint + format check
npm run biome:fix       # Auto-fix formatting
npx vitest run --exclude tests/install   # Run all tests except install checks
```

> The pre-commit hook runs `types:check`, `biome:check`, and Vitest automatically.

### Creating new modules (pattern cloning — preferred over writing from scratch)

```bash
# Clone an existing module, replacing all name references
npm run make:clone -- <source_dir> <dest_dir> <OldName> <NewName>

# Example: create a "post" module from "item"
npm run make:clone -- src/express/modules/item src/express/modules/post Item Post
```

After cloning an express module, register the new routes in src/express/routes.ts:

```typescript
await importAndUse("./modules/post/postRoutes");
```

After cloning a react module, register the new routes in src/react/routes.tsx:

```tsx
import { postRoutes } from "./components/post/index";

/* ... */

const routes: RouteObject[] = [
  {
    /* ... */
    children: [
      /* ... */
      ...postRoutes,
    ],
  },
];
```

> Always prefer `make:clone` over writing modules from scratch. It replicates your actual patterns.

### Cleanup

```bash
npm run make:purge                       # Remove example modules (item, post, auth, user)
npm run make:purge -- --keep-auth        # Remove items but keep auth and user
npm run install:check                    # Verify .env and database file are accessible
```

### Production

```bash
docker compose -f compose.prod.yaml up --build   # Build + start prod containers
# Prod sets NODE_ENV=production, runs: npm run build && npm start
```

---

## Architecture — key decisions

### One server (not two)

There is **one** Node process serving both the Express API and the React frontend via SSR. `server.ts` is the single entry point. Vite runs in middleware mode embedded inside Express — there is no separate Vite dev server to proxy.

- API routes (`/api/*`) are handled by Express.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rocambille/start-express-react](https://github.com/rocambille/start-express-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
