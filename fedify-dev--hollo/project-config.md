---
trigger: always_on
description: Hollo coding guidelines for AI assistants
---

Hollo coding guidelines for AI assistants
==========================================

Hollo is a federated single-user microblogging software powered by [Fedify].
It implements the [ActivityPub] protocol for federation with other platforms
(like Mastodon, Misskey, etc.) and provides Mastodon-compatible APIs for
client integration.

[Fedify]: https://fedify.dev/
[ActivityPub]: https://www.w3.org/TR/activitypub/


Project overview
----------------

 -  *Technology stack*: TypeScript (ESNext), Hono.js (web framework with JSX),
    Drizzle ORM, PostgreSQL
 -  *Package manager*: pnpm only (enforced via `packageManager` field)
 -  *Runtime*: Node.js with tsx
 -  *License*: GNU Affero General Public License v3 (AGPL-3.0)
 -  *Structure*: Single-user microblogging platform with federation
    capabilities
 -  *API*: Implements Mastodon-compatible APIs for client integration


Directory structure
-------------------

~~~~
hollo/
├── bin/                          # Entry point scripts
│   ├── server.ts                 # Main server entry point
│   └── routes.ts                 # Debug utility to list all routes
│
├── src/                          # Main application source
│   ├── api/                      # Mastodon-compatible REST APIs
│   │   ├── v1/                   # API v1 endpoints
│   │   └── v2/                   # API v2 endpoints (search, notifications)
│   │
│   ├── components/               # Hono JSX components (server-rendered)
│   │
│   ├── entities/                 # Entity serialization (DB → API response)
│   │
│   ├── federation/               # ActivityPub implementation via Fedify
│   │   ├── index.ts              # Federation setup and inbox listeners
│   │   ├── actor.ts              # Actor dispatchers (Person, Organization)
│   │   ├── inbox.ts              # Inbox activity handlers
│   │   ├── objects.ts            # Object dispatchers (Note, Article, Question)
│   │   └── ...
│   │
│   ├── import/                   # Background data import (from Mastodon exports)
│   │   ├── processors.ts         # Import item processors
│   │   └── worker.ts             # Background job worker
│   │
│   ├── oauth/                    # OAuth 2.0 / OpenID Connect implementation
│   │   ├── constants.ts          # Token sizes, expiry times
│   │   ├── middleware.ts         # Auth middleware (tokenRequired, scopeRequired)
│   │   └── endpoints/            # OAuth endpoints (metadata, revoke, userinfo)
│   │
│   ├── pages/                    # Web UI pages (Hono JSX)
│   │
│   ├── public/                   # Static assets (CSS, favicons)
│   │
│   ├── index.tsx                 # Main Hono app composition
│   ├── schema.ts                 # Drizzle ORM database schema
│   ├── db.ts                     # Database connection
│   └── ...
│
├── scripts/                      # Utility scripts
│   └── rebuild-timelines.ts      # Rebuild all home timelines
│
├── tests/                        # Test utilities and fixtures
│   ├── helpers.ts                # Database cleanup, test fixtures
│   └── helpers/                  # OAuth and web test utilities
│
├── drizzle/                      # Database migrations (SQL files)
│
└── docs/                         # Documentation site (Astro/Starlight)
~~~~


Key architectural components
----------------------------

### Core layers

 -  *API layer* (*src/api/*): Implements Mastodon-compatible REST APIs
    (v1 and v2)
 -  *Federation* (*src/federation/*): ActivityPub implementation using Fedify
 -  *Database* (*src/db.ts* and *src/schema.ts*): PostgreSQL with Drizzle ORM
 -  *OAuth* (*src/oauth/*): OAuth 2.0 with OpenID Connect support
 -  *Components* (*src/components/*): Hono JSX components for server-rendered UI
 -  *Entities* (*src/entities/*): Transform database records to Mastodon API
    responses
 -  *Pages* (*src/pages/*): Web UI pages (profile, setup, dashboard, etc.)
 -  *Import system* (*src/import/*): Background job processing for data imports

### Key files

| File                        | Purpose                                   |
| --------------------------- | ----------------------------------------- |
| *src/index.tsx*             | Main Hono app that composes all routes    |
| *src/schema.ts*             | Complete Drizzle ORM schema (~1300 lines) |
| *src/federation/index.ts*   | Federation setup with inbox listeners     |
| *src/oauth/middleware.ts*   | Authentication middleware                 |
| *src/entities/status.ts*    | Status entity serialization               |


Technology stack
----------------

### Core dependencies

| Category      | Package           | Version | Purpose                        |
| ------------- | ----------------- | ------- | ------------------------------ |
| Runtime       | tsx               | ^4.21   | TypeScript executor            |
| Web framework | hono              | ^4.11   | HTTP server with JSX support   |
| Federation    | @fedify/fedify    | ~1.10   | ActivityPub implementation     |
| Database      | drizzle-orm       | ^0.45   | ORM for PostgreSQL             |
| Database      | postgres          | ^3.4    | PostgreSQL client              |
| Storage       | flydrive          | ^1.3    | S3/filesystem abstraction      |
| Validation    | zod               | ^4.2    | Schema validation (v4, not v3) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fedify-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
