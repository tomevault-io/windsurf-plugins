---
trigger: always_on
description: This file provides guidance to Claude Code when working with the OpenMeet API.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with the OpenMeet API.

## Project Overview

OpenMeet API is a NestJS-based multi-tenant backend for event management and community building. It integrates with the AT Protocol for decentralized identity, supports multiple OAuth providers, and provides real-time event streaming via RabbitMQ.

## Tech Stack

- **Framework**: NestJS with TypeScript
- **Database**: PostgreSQL with TypeORM (multi-tenant via schemas)
- **Cache**: Redis/ElastiCache for sessions and caching
- **Queue**: RabbitMQ for async event processing
- **Auth**: JWT + multiple OAuth providers (AT Protocol/Bluesky, Google, GitHub)
- **AT Protocol**: @atproto/* packages for decentralized identity

## Terminology: AT Protocol Transition

**Long-term goal:** Transition from "Bluesky/bsky" terminology to "AT Protocol/atproto" terminology throughout the codebase.

**Why:** AT Protocol is the underlying decentralized protocol; Bluesky is just one application built on it. OpenMeet integrates with the protocol itself, not specifically with Bluesky the app. Using protocol-level terminology:
- Better reflects what we're actually integrating with
- Avoids confusion as other apps join the AT Protocol network
- Aligns with the @atproto/* packages we use

**When making changes:**
- **New code**: Use "atproto" or "AT Protocol" terminology (e.g., `atproto-auth`, `AtprotoService`)
- **Refactoring**: Take opportunities to rename `bluesky`/`bsky` → `atproto` when touching related code
- **Commits**: Use "atproto" in commit messages for new AT Protocol work
- **Don't**: Do wholesale renames just for terminology—combine with functional changes

**Current state** (to be migrated over time):
- `src/auth-bluesky/` → eventually `src/auth-atproto/`
- `src/bluesky/` → eventually `src/atproto/`
- `BLUESKY_KEY_*` env vars → eventually `ATPROTO_KEY_*`

## AT Protocol Data Architecture: Public vs Private

OpenMeet is a **permissioned data appview**. This is the ATProto-recommended pattern where identity is decentralized (DIDs) but authorization is centralized (server-side RBAC).

**The core rule: public data goes on-protocol, private data stays in our database.**

| Data | Where It Lives | Why |
|------|---------------|-----|
| Public events | User's PDS + OpenMeet DB | Discoverable, federable, user-owned |
| Private events | OpenMeet DB only | Never published to any PDS — invisible to other appviews |
| RSVPs to public events | User's PDS + OpenMeet DB | User's social activity, portable |
| Attendee lists for private events | OpenMeet DB only | Membership info is access-controlled |
| Group membership/permissions | OpenMeet DB only | Authorization state, not identity |
| User identity (DID, handle) | PDS + OpenMeet DB | Decentralized, portable |

**Why not encrypt private data on PDS?** ATProto repos are public by design. Encryption on PDS leaks metadata (existence of records) and requires key management infrastructure (Keyhive) that doesn't exist yet. Server-side access control is simpler, sufficient, and what the ATProto team recommends for this use case.

**When building features:**
- If the data should be visible to other ATProto apps → publish to PDS
- If the data is gated by group membership, visibility settings, or permissions → keep in DB only
- Events with `visibility: 'private'` should never get `atprotoUri`/`atprotoRkey` fields populated

## Development Commands

```bash
# For general development tasks, we use docker compose to run a full environment
# to manage just the api with live reloading

docker compose -f docker-compose-dev.yml  up --build -d api 
docker compose -f docker-compose-dev.yml logs api -f
docker compose -f docker-compose-dev.yml down api



# For Local development
# Start development server 
npm run start:dev # we generally use the docker setup above

# Run unit tests
npm run test:local

# Run specific test file
npm run test -- path/to/file.spec.ts

# run the full 15 minute e2e test suite, specify files to test if we can to improve spped
npm run test:e2e

# Lint and fix
npm run lint -- --fix

# Type check (no npm run typecheck available)
npx vue-tsc --noEmit   # Not applicable here - use tsc
npx tsc --noEmit

# Build
npm run build
```

### Database Commands

```bash

# we use .env to point at the correct database, linking .env-local to .env for local environments, but linking .env-dev or .env-prod when we need to update those envs.  maybe we need to transition this to a job that runs migrations in prod/dev instead of using my local host to do it

# Run migrations
ln -sf .env-local .env
npm run migration:run:tenants

# Reset database to empty (dev only)
npm run migration:reset

# Generate new migration
# get date stamp in nanoseconds
# create a src/database/migrations file and look at a few existing examples for the patterns to follow
```

## Architecture

```
design-notes/             # design docs to help us keep the project on track and steer long term development
grafana/                  # various dashboards that we use for grafana.  should probably move to infrastructure
test/                     # e2e tests for API and related services
src/
├── auth/                 # Core authentication
├── auth-bluesky/         # Bluesky OAuth integration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenMeet-Team/openmeet-api](https://github.com/OpenMeet-Team/openmeet-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
