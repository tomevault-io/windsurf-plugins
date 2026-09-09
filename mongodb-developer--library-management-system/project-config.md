---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

A MEAN-stack (MongoDB, Express, Angular, Node.js) library app used in MongoDB
Developer Days workshops. The server (`server/`) exposes a REST API backed by
MongoDB Atlas; the client (`client/`) is an Angular SPA that consumes it.

## Build and test commands

```bash
npm install                       # root: installs husky for the pre-commit hook
cd server && npm install          # server dependencies
cd client && npm install          # client dependencies

cd server && npm run build:watch  # compile TypeScript, watch mode
cd server && npm run serve        # run compiled server with nodemon
cd server && npm start            # build:watch + serve together (typical dev loop)
cd server && npm run lint         # eslint src/
cd server && npm run api-test     # mocha, requires a running server + MongoDB
cd server && npm test             # starts the server, waits for it, then runs api-test

cd client && npm start            # ng serve on http://localhost:4200
cd client && npm run build        # ng build
cd client && npm test             # ng test (Karma)
```

The pre-commit hook (`.husky/pre-commit`) brings up `.devcontainer/docker-compose.yml`
(a local MongoDB Atlas Local container) in the background, then runs `npm run lint`
and `npm test` inside `server/`. Docker must be running locally for `git commit` to
succeed outside of Codespaces/the devcontainer.

Smoke check after a change:

1. Run the commands above to start `server` (port 5400) and `client` (port 4200).
2. Open http://localhost:4200 and confirm the book list loads.
3. Log in, borrow or reserve a book, and confirm the change is reflected in the UI.

## Project structure

```
server/src/
  server.ts            # Express app entry point, route mounting
  database.ts          # MongoDB client/connection, collection handles
  load-env-vars.ts      # dotenv bootstrap, imported first in every entry point
  models/              # TypeScript interfaces for each collection's documents
  routes/              # Express routers (one per resource) plus *.spec.ts API tests
  controllers/         # Request handlers used by routes/
  utils/               # Auth middleware, shared test helpers
  embeddings/           # Pluggable embedding providers (OpenAI, Google Vertex, a
                         # serverless endpoint) selected via EMBEDDINGS_SOURCE
  search-indexing/      # Standalone scripts that create/update MongoDB Search and
                         # Vector Search indexes on the books collection (workshop labs)
  indexing/             # Standalone script demonstrating a compound index via .explain()
client/src/app/         # Angular components, services, routing
migrations/             # One-off data migration scripts (run manually with Node)
.devcontainer/          # Codespaces/Dev Containers setup: local Atlas container,
                         # sample data import, port forwarding, best-effort
                         # created/started lifecycle telemetry (track.sh)
```

Notable files:

- [server/src/database.ts](server/src/database.ts) — throws a descriptive error if
  `DATABASE_URI` is unset or still the placeholder value; read this first when the
  server won't start.
- [server/src/models/issue-detail.ts](server/src/models/issue-detail.ts) — a single
  collection (`issueDetails`) holds both borrowed-book and reservation records,
  distinguished by `recordType`.
- [server/src/search-indexing/*.ts](server/src/search-indexing) — each file is a
  runnable workshop step (`node --loader ts-node/esm src/search-indexing/<file>.ts`
  after building) that creates one MongoDB Search/MongoDB Vector Search index variant. Several
  contain `<REPLACE_WITH_...>` placeholders left in on purpose for the workshop
  exercise — do not "fix" them as bugs.

## API overview

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/authors/:authorId` | Fetch an author and their books |
| GET | `/books` | List books |
| GET | `/books/search` | Full-text/vector search over books |
| GET | `/books/:bookId` | Fetch a single book |
| POST | `/books` | Create a book (admin) |
| PUT | `/books/:bookId` | Update a book (admin) |
| DELETE | `/books/:bookId` | Delete a book (admin) |
| GET | `/books/:bookId/reviews` | List reviews for a book |
| POST | `/books/:bookId/reviews` | Add a review (authenticated) |
| GET | `/books/:bookId/reviews/:reviewId` | Fetch a single review |
| GET | `/users/login/:username?` | Log in / issue a token |
| GET | `/reservations` | List the current user's reservations |
| GET | `/reservations/page` | Paginated reservations (admin) |
| GET | `/reservations/:reservationId` | Fetch a single reservation |
| POST | `/reservations/:bookId` | Reserve a book |
| DELETE | `/reservations/:bookId` | Cancel a reservation |
| GET | `/reservations/user/:userId` | List a user's reservations (admin) |
| GET | `/borrow` | List the current user's borrowed books |
| GET | `/borrow/history` | Borrow history |
| GET | `/borrow/page` | Paginated borrow records (admin) |
| POST | `/borrow/:bookId/:userId` | Borrow a book (admin) |
| POST | `/borrow/:bookId/:userId/return` | Return a book (admin) |

## Environment variables and configuration

| Name | Required | Example | Description |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mongodb-developer/library-management-system](https://github.com/mongodb-developer/library-management-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
