---
trigger: always_on
description: This is a monorepo for Couchers.org, a non-profit couch surfing platform. Users sign up and can be hosts (offering their couch/spare room to travelers), surfers (requesting to stay with hosts), community members (attending events, joining discussions, and building local communities), or any combination. Core features include user profiles with hosting preferences, sending and responding to couch requests, messaging between users, community features like events and discussions organized by local 
---

# Claude Code Instructions for Couchers

## Summary

This is a monorepo for Couchers.org, a non-profit couch surfing platform. Users sign up and can be hosts (offering their couch/spare room to travelers), surfers (requesting to stay with hosts), community members (attending events, joining discussions, and building local communities), or any combination. Core features include user profiles with hosting preferences, sending and responding to couch requests, messaging between users, community features like events and discussions organized by local communities, and a reference system for building trust. The platform emphasizes safety, community building, and keeping the service free and community-owned.

## Git

- The main branch is `develop` (not `main`)

## Repository Structure

- `/app/backend` - Python backend (gRPC, SQLAlchemy, PostgreSQL/PostGIS). See `/app/backend/readme.md` for more details
- `/app/web` - Next.js web frontend
- `/app/mobile` - React Native Expo mobile app (uses `npm`, not `yarn`). See `/app/mobile/README.md` for setup instructions
- `/app/proto` - Protocol buffer definitions shared across services
- `/docs` - Documentation

## Backend Commands

All backend commands should be run from `/app/backend`:

```bash
# Format code (run after any backend change)
make format
# or: uv run ruff check --fix . && uv run ruff format

# Run tests (you can use additional commands)
uv run pytest
# if you get an error about the database not being up, ask the user to start the testing database

# Generate protobuf files (after changing .proto files)
make protos

# Type checking
make mypy
```

## Key Conventions

### Backend (Python)
- Uses `uv` for dependency management
- SQLAlchemy 2.0 with mapped_column style
- gRPC for API (defined in `/app/proto`)
- Background jobs in `couchers/jobs/handlers.py`
- Notifications system in `couchers/notifications/`
- Always run `make format` and `make mypy` after modifying backend code. mypy MUST pass — a failing mypy is never acceptable, so fix it before moving on (don't dismiss errors as "pre-existing")
- If mypy or tests fail with import errors or missing symbols from generated proto modules (`couchers.proto.*` — e.g. a message type that exists in a `.proto` source but not in the generated `*_pb2.py`), your locally generated protos are stale: run `make protos` to regenerate them, then re-check
- NEVER try-catch an exception and silently throw it away or just log it. By and large you don't need to wrap code in try-catch blocks, we already handle exceptions
- Use `enum.auto()` for all enums (except in the rare case that they are inherently ordinal and we use that order in business logic)
- Put relationships and constraints at the end of models
- For text columns, don't use fixed-length strings in models, don't use `Text`. Use `String`
- All database constraints go in models
- When adding environment variables for the backend, carefully add them to `backend.dev.env` and into the test environment
- Imports always occur at the top of the file. The two exceptions are when this is required during type checking or in tests that really require inline imports
- Do not use `session.get(...)`. Use `session.execute(select(...))` instead
- For URLs, use `from couchers import urls` and then `urls.whatever()`
- Avoid inline imports whenever possible
- To filter out invisible users (deleted/banned/blocked), use the helper functions from `couchers.sql`: `where(users_visible(context))` when User is already joined, `where(users_column_visible(context, column))` when you have a user_id column, or `where(users_visible_to_each_other(user1, user2))` for mutual visibility. Never use `User.is_visible` directly in queries

### Web (TypeScript/React)
- Uses `nvm` for node version management
- Uses `yarn` (not npm) - run dev server with `yarn start` (not Docker)
- Run linting with `yarn lint` and auto-fix with `yarn lint:fix`
- Run tests with `yarn test`
- Run linting AND formatting with `yarn format`
- Import aliases: use `components/` not `../../../components/`, `routes` not `../../../routes`
- Import multiple MUI icons together: `import { Favorite, Star, Public } from "@mui/icons-material"` instead of separate imports
- No `any` types - explicitly type mock mutations (e.g., `UseMutationResult<...>`)
- Use StyledLink or next/link for routing - NOT MUI `Link`
- Type definitions should always go at the top of the file below the imports
- **IMPORTANT**: When using Material-UI components (Button, Chip, MenuItem, etc.) with the `href` prop for internal navigation, ALWAYS use `component={Link}` instead of `component="a"` to preserve locale prefixes. Import Link from `next/link`
- Remove extra unnecessary style declarations - don't repeat anything that's already a default of MUI or the theme

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Couchers-org/couchers](https://github.com/Couchers-org/couchers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
