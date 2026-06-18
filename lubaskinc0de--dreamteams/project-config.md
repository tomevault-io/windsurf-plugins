---
trigger: always_on
description: Guidance for AI agents working in this repository.
---

# AGENTS.md

Guidance for AI agents working in this repository.

## Project Overview

DreamTeams is a competition management platform for hackathons, olympiads, and team-based events. It has a Nuxt frontend, a FastAPI main API, and a separate exporter service for asynchronous CSV exports.

The backend follows strict Clean Architecture and is split into bounded contexts under `src/`:

- `dreamteams` - main context: users, organizers, participants, competitions, applications, invites, tags, and profiles.
- `dreamteams_exporter` - export context: export jobs, NATS processing, HTTP reads from the main API, and S3-compatible CSV storage.
- `dreamteams_common` - shared primitives only: `AppError`, `@interactor`, `UoW`, clock, logger aliases, structlog setup, and OpenTelemetry setup.

The main and exporter contexts must not import each other. They communicate over HTTP and NATS.

## Essential Commands

Use `just` recipes unless a task needs a narrower command.

```bash
just up                         # Start the local stack in the foreground
just up-silent                  # Start the local stack detached
just up-db                      # Start PostgreSQL only
just down                       # Stop app-facing containers
just down-all                   # Stop all compose services
just clear                      # Reset app DB and selected volumes
just clear-all                  # Remove all compose volumes
just test                       # Run pytest with xdist
just test-cov                   # Run pytest with coverage
just test-unit                  # Run unit tests
just lint                       # Run ruff format/check --fix, mypy, import-linter, typos
just dev-environment            # Install Python dev extras and frontend dependencies
just build-frontend             # Generate static frontend assets
just docs                       # Serve MkDocs documentation
just generate-migration "Name"  # Generate Alembic migration
```

`just test` runs pytest locally with xdist. It does not start a Docker test compose file.

## Architecture Rules

Import boundaries are enforced by `.importlinter`.

Main context:

- `dreamteams.entities` must not import application, adapters, presentation, or bootstrap.
- `dreamteams.application` must not import adapters, presentation, or bootstrap.
- `dreamteams.adapters` must not import presentation or bootstrap.
- `dreamteams.presentation` must not import bootstrap.

Exporter context follows the same layer rules.

Cross-context rules:

- `dreamteams` must not import `dreamteams_exporter`.
- `dreamteams_exporter` must not import `dreamteams`.
- `dreamteams_common` must not import either bounded context.

If an import-linter failure appears, fix it at the protocol, DTO, or transport boundary. Prefer duplicating a tiny transport shape in each context over coupling contexts through Python imports.

## Backend Patterns

- Use immutable dataclass interactors decorated with `@interactor`.
- Keep use cases in `application/{feature}/`.
- Define ports in the application layer and implement them in adapters.
- Do not use SQLAlchemy models directly from application code.
- Route database writes through the Unit of Work protocol.
- Register interactors and adapters in Dishka providers under `bootstrap/di/providers/`.
- Keep domain-specific code out of `dreamteams_common`.
- Use structured DTOs and value objects instead of ad hoc dictionaries when the codebase already has a type for the shape.

Application-layer organization:

- Group code by business capability or use-case family, not by technical pattern.
- Keep orchestration in interactors: load through gateways, delegate decisions to domain objects, persist through UoW, publish events, return DTOs.
- Keep infrastructure out of interactors; add or extend gateway protocols when a use case needs persistence, cache, storage, broker, auth, or HTTP access.
- Put request/response DTOs near the use case when they are feature-specific; put shared DTOs under `application/common/dto`.

Domain-model expectations:

- Prefer rich entities and value objects over passive records plus scattered procedural checks.
- Put invariants where the data lives: competition schedule/team-size rules in competition objects, application status transitions in application objects, invite lifecycle rules in invite objects, profile validation in participant/organizer value objects.
- Use domain factories to construct valid entities and prevent invalid state from entering later layers.
- Raise explicit domain/application errors rather than generic exceptions for business-rule failures.
- Unit-test domain behavior directly before adding broader integration coverage.

## Database and Persistence

The main application uses PostgreSQL with SQLAlchemy 2.0 imperative mappings.

Important model groups:

- `users` and `auth_user` for internal users and Authentik subject linkage.
- `organizer` and `organizer_invite` for organizer registration and invite lifecycle.
- `participants`, `participant_skills`, and `participant_contacts` for participant profiles.
- `competitions`, `competition_tags`, `competition_tag_links`, `competition_tracks`, and `milestones` for competition publishing and discovery.
- `application_forms` for custom per-competition forms.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lubaskinc0de/dreamteams](https://github.com/lubaskinc0de/dreamteams) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
