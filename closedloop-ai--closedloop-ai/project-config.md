---
trigger: always_on
description: `apps/` contains deployable surfaces: `app` (product UI, port 3000), `api` (BFF/server, 3002), `web` (marketing, 3001), plus `docs`, `email`, `storybook`, `mcp`, `relay`, and `studio`. Shared code lives in `packages/*` and is imported as `@repo/<name>`; database schema and migrations live in `packages/database`. Repo documentation is in `docs/`, and CI/workflow automation is in `.github/`. `apps/api` is deployed on Vercel serverless functions, so route code must not rely on process-local memory,
---

# Repository Guidelines

## Project Structure & Module Organization
`apps/` contains deployable surfaces: `app` (product UI, port 3000), `api` (BFF/server, 3002), `web` (marketing, 3001), plus `docs`, `email`, `storybook`, `mcp`, `relay`, and `studio`. Shared code lives in `packages/*` and is imported as `@repo/<name>`; database schema and migrations live in `packages/database`. Repo documentation is in `docs/`, and CI/workflow automation is in `.github/`. `apps/api` is deployed on Vercel serverless functions, so route code must not rely on process-local memory, singleton state, or long-lived in-process caches for correctness. Keep data flow layered: `apps/app` should call `apps/api`, and only the API should touch `@repo/database`.

## Cross-Repo Compatibility Requirements
Changes in this repo must not assume another repo (for example `closedloop-electron`) is upgraded at the same time.

- Treat all cross-repo contracts (desktop gateway payloads, relay events, error reasons, callback semantics) as version-skewed.
- New fields must be additive and optional; missing/unknown values must degrade gracefully to safe defaults.
- For optional cross-repo payload fields, preserve omission when a value is absent. Do not serialize absent optional fields as `null` unless the receiving contract explicitly declares that field nullable and old clients are known to accept it.
- When external payload fields are renamed, keep the previous field accepted as a compatibility alias until a human explicitly approves removing the shim, as long as the server can map it to the new behavior safely.
- Never crash, throw unhandled errors, or block core flows solely because a peer repo is on an older/newer version.
- For behavior/classification changes, include a backward-compatible fallback path (for example, map unknown reasons to generic `launch_failed`).
- Update tests to cover both:
  - new/expected contract shape
  - old or unknown contract shape (graceful fallback)

## Build, Test, and Development Commands
Use Node 20+ with `pnpm`.

- `pnpm install` installs workspace dependencies and generates the Prisma client.
- `docker compose up -d` starts local PostgreSQL; `just db-start` is the lightweight alternative.
- `pnpm dev` runs the Turborepo dev graph; `just dev` starts the main local stack (`app`, `api`, `mcp`).
- `pnpm turbo dev --filter=app --filter=api` focuses on the primary product surfaces.
- `pnpm build`, `pnpm typecheck`, `pnpm lint`, and `pnpm test` run workspace-wide checks.
- `pnpm migrate` or `just db-migrate name=my_change` creates/applies Prisma migrations.
- For Prisma schema changes, generate migrations with `prisma migrate dev` or `prisma migrate dev --create-only`; only hand-edit the generated SQL for constructs Prisma cannot express, such as partial unique indexes.
- For pre-commit validation, prefer `just build` plus `git diff --check` when the change is ready for final verification. Do not also run separate workspace/app test, typecheck, or lint commands unless you are isolating a specific failure, need a faster focused loop while debugging, or the user explicitly requests those commands.

## Dockerized Workspace Apps
Some apps, including `apps/mcp` and `apps/relay`, build from narrow Docker contexts instead of the full monorepo. When adding or changing any `@repo/*` import or `workspace:*` dependency in a Dockerized app, update that app's Dockerfile in the same change.

- Copy every required workspace package into the builder stage before `pnpm install`, including transitive workspace dependencies needed by that package.
- Copy package manifests for those workspace packages into the runtime stage before `pnpm install --prod`.
- If runtime executes TypeScript with `tsx` or uses deep imports such as `@repo/api/src/...`, copy the needed `src/` or built `dist/` output into the runtime image so module resolution works after deploy.
- Validate both the builder target and full image for the changed app, for example `docker buildx build --file apps/relay/Dockerfile --target builder .` and `docker buildx build --file apps/relay/Dockerfile .`. A local `pnpm build` or `pnpm typecheck` is not enough for these Dockerized apps because it does not prove the container has the same workspace package files.

## Coding Style & Naming Conventions
TypeScript and ESM are standard across the repo. Formatting and linting are enforced by Biome with Ultracite presets; run `pnpm lint:fix` before opening a PR. Follow the existing 2-space indentation, prefer `type` aliases when practical, and keep `@repo/*` imports ahead of local alias imports. File names are typically kebab-case (`pull-request-status-badge.tsx`), while exported React components and types use PascalCase. In `apps/api`, keep route handlers thin and move business logic into nearby `service.ts` modules.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [closedloop-ai/closedloop-ai](https://github.com/closedloop-ai/closedloop-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
