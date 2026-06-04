---
trigger: always_on
description: - Commit after every logical set of changes.
---

# Prismedia Repository Contract

## Commit & Changelog Policy

- Commit after every logical set of changes.
- Keep commits small, reviewable, and intentionally scoped.
- With every release-note-worthy change, update `CHANGELOG.md` under `## [Unreleased]`.
- Keep changelog entries user-facing and high level. Internal cleanup belongs in git history, not the changelog.
- Use Keep a Changelog sections: `What's New`, `Added`, `Changed`, `Fixed`, `Removed`, and `Docs`.
- The root `package.json` version is the build version and the single source of truth. All workspace package versions must match it exactly.
- Do not change the build version while publishing a channel image. Move the version forward when a new body of work starts, then let dev, alpha, beta, and release images publish that decided version.
- Suggested commit style:
  - `chore: bootstrap workspace`
  - `docs: define repo contract`
  - `feat(web): add media library shell`
  - `feat(api): add health and jobs routes`
  - `fix(worker): stabilize queue startup`

## Versioning

- Prismedia starts at `1.0.0`.
- Versions are plain `X.Y.Z`; do not use development suffixes.
- MAJOR: breaking API changes, schema changes that require user action, or config format changes.
- MINOR: new features, new API endpoints, or new UI views.
- PATCH: bug fixes, UI tweaks, dependency updates, and documentation.
- Docker builds run `pnpm release:check`, which verifies changelog structure and workspace package version alignment.

## Release Channels

- Every push to `main` builds only `ghcr.io/pauljoda/prismedia:dev`.
- Manual channel publishing is handled by `.github/workflows/publish-channel.yml`.
- The manual workflow accepts `alpha`, `beta`, or `release`.
- `alpha` publishes `alpha`, `alpha-<version>`, and `alpha-<version>-<short-sha>`.
- `beta` publishes `beta`, `beta-<version>`, and `beta-<version>-<short-sha>`.
- `release` publishes `release`, `release-<version>`, `release-<version>-<short-sha>`, and `latest`.
- Publishing a channel image never edits package versions, rewrites changelog headings, creates git tags, or commits release artifacts.

## Product

Prismedia is a private media library for self-hosted collections. It is video-first, but images, galleries, books, comics, audio, performers, studios, tags, and collections are first-class library entities. It is optimized for a single trusted user or household on a private LAN and ships as a Docker image.

Stash compatibility exists for plugin and metadata workflows. Prismedia should not be framed as a replacement for another app, and third-party schemas should not define Prismedia's persistence model.

## Project Structure

```text
apps/web-svelte/       Svelte frontend only. Built as static assets and served by the .NET API.
apps/backend/          .NET API, application/domain/infrastructure layers, EF Core persistence, and .NET worker.

packages/contracts/    Frontend TypeScript constants, media helpers, and plugin protocol types.
packages/media-core/   File discovery, fingerprint, and scan primitives.
packages/plugins/      Plugin runtime helpers and contracts.
packages/stash-compat/ Stash-compatible scraper and StashBox protocol helpers.
packages/ui-svelte/    Shared Svelte design tokens and UI primitives.

infra/docker/          Dockerfiles and dev compose stack.
scripts/release/       Version and changelog validation tooling.
docs/                  Architecture and design language docs.
```

## Architecture

- Monorepo with `pnpm` workspaces and `turbo`.
- Runtime processes: .NET API/HTTP ingress, static Svelte frontend assets, and the .NET worker.
- PostgreSQL 16 is the sole stateful dependency for application data and queue/job state.
- Public HTTP contracts live in the .NET backend and are consumed by the generated Svelte client under `apps/web-svelte/src/lib/api/generated`.
- Backend work must follow `docs/backend-architecture-contract.md`: Clean Architecture, DDD-lite domain behavior, CQRS-lite use cases, EF Core as infrastructure persistence, DTO API boundaries, and generated frontend clients.

## Key Decisions

1. .NET API + static Svelte UI: the .NET API owns endpoints, persistence, and server orchestration. Svelte is a frontend client only.
2. PostgreSQL + EF Core: typed schema and EF migrations are managed from `apps/backend/src/Prismedia.Infrastructure/Persistence`.
3. .NET background worker: scan, probe, thumbnail, sprite, HLS, and metadata work runs in `apps/backend/src/Prismedia.Worker`.
4. HLS streaming: videos are transcoded to HLS on demand via ffmpeg and served by the .NET API.
5. Typed contracts: .NET contracts are the server source of truth. The frontend should prefer generated OpenAPI types.
6. No global EntityGraph: relationship links are EF persistence structures for bounded domain slices and read projections.

## Database

- Schema is defined by EF Core entity mappings in `apps/backend/src/Prismedia.Infrastructure/Persistence`.
- Core entities: media entities, performers, studios, tags, fingerprints, library roots, settings, sources, files, and job runs.
- EF migration files live under `apps/backend/src/Prismedia.Infrastructure/Persistence/Migrations` and are applied by the .NET runtime on startup.
- Adding a schema change:
  1. Edit the EF Core entity/model mapping.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pauljoda/Prismedia](https://github.com/pauljoda/Prismedia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
