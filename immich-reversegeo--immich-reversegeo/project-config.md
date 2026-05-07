---
trigger: always_on
description: **immich-reversegeo** is a self-hosted companion service (ASP.NET Core + Blazor Server) that improves Immich location metadata. It reads GPS-tagged assets from the Immich PostgreSQL database, resolves country and administrative areas from Overture Maps data, uses bundled Overture airport infrastructure where helpful, and writes city/state/country back to the `asset_exif` table.
---

# AGENTS.md

## Project Overview

**immich-reversegeo** is a self-hosted companion service (ASP.NET Core + Blazor Server) that improves Immich location metadata. It reads GPS-tagged assets from the Immich PostgreSQL database, resolves country and administrative areas from Overture Maps data, uses bundled Overture airport infrastructure where helpful, and writes city/state/country back to the `asset_exif` table.

## Repo Conventions

- In user-facing copy, use the product name **Immich ReverseGeo**. Keep lowercase `immich-reversegeo` only for technical slugs: URLs, repo names, Docker tags, image names.
- The service defaults to port `8080` in Docker and `5122` for local dev.
- The Docker image name is `immich-reversegeo` (or `ghcr.io/immich-reversegeo/immich-reversegeo`).
- Keep public docs and README copy concise and plain. Avoid hype, filler, and over-marketing language.
- Write public docs for end users and self-hosters, not for contributors or maintainers.
- Prefer plain-language explanations over internal data-model terms. Introduce terms like `division_area`, `subtype`, or `admin_level` only when they are necessary, and explain them in user-facing language.
- Public docs should explain user goals, limits, and recommended workflow first. Technical implementation detail should only appear when it helps the reader make a decision.
- When documenting advanced features, include what the feature can fix, what it cannot fix, and how to validate behavior with the app before changing settings.
- Prefer concrete troubleshooting steps over abstract descriptions. If the recommended workflow is “use Lookup first, inspect the result, then decide,” say that directly.
- Public docs use **Zensical** with the existing `mkdocs.yml` compatibility path, not a separate static-site generator setup. Main public routes should live under the website docs.
- Public website content lives under `docs/website/`. Maintainer-only docs live under `docs/maintainer/`.
- Keep the README product-first. Public setup and product documentation should usually live in `docs/website/` or `CONTRIBUTING.md`, not in the README.
- The target audience for the public website is end users and self-hosters, not contributors. Public setup docs should be Docker-first unless there is a strong reason not to.
- Local developer setup belongs in `CONTRIBUTING.md` or maintainer/developer docs, not in public Getting Started pages.
- When a change affects user-visible behavior, setup, settings, processing expectations, or other public-facing workflows, update the relevant docs in the same change unless the deferral is explicit.
- Keep a technical changelog in `CHANGELOG.md` and a user-facing release summary in `docs/website/changelog.md`. When one is updated for a release, keep the other in sync and link between them.
- Config lives under `/config` at runtime; geodata and caches live under `/data`. These are separate volumes so config (secrets) and data (downloadable/regenerable) are mounted independently.
- Put non-source local output under `_out/` or `localdata/` — both are gitignored. Do not commit them.
- Keep `.planning/` local-only and gitignored. Do not commit planning or spec work.
- Treat untracked local tooling folders such as `.claude/`, `.superpowers/`, `.playwright-mcp/`, `.vs/`, and `localdata/` carefully. Do not commit or clean them up unless explicitly requested.
- Prefer [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/) for commit messages. Common types in this repo include `feat`, `fix`, `docs`, `test`, `ci`, `chore`, and `cleanup`.

---

## Code Style

- **Always use braces** for `if`, `else`, `foreach`, `while`, `for` — even single-statement bodies. No braceless one-liners.
- Allman style: opening brace on its own line for method/class bodies; same line is acceptable for short inline catch blocks (`catch (X) { return; }`).
- Expression-body members (`=>`) are fine for trivial one-liners (properties, delegates, simple methods).

## Tech Stack

.NET 10, ASP.NET Core, Blazor Server (Interactive), Npgsql, NetTopologySuite, DuckDB, MSTest SDK on Microsoft.Testing.Platform

---

## Commands

```bash
# Run locally → http://localhost:5122
npm run start

# Stop the running dev app before running tests (Windows locks the exe)
# Run all tests
npm run test

# Run integration tests explicitly
npm run test:integration

# Build docs / exports
npm run docs:build
npm run export:airports
npm run export:country-divisions

# Build Docker image
docker build -t immich-reversegeo .

# Deploy (add to existing Immich compose)
docker compose up -d
```

Test notes:
- This repo uses MSTest SDK on Microsoft.Testing.Platform v2 via `global.json`.
- Prefer `npm run test` / `npm run test:integration` for normal runs.
- If you call `dotnet test` directly, use `--project <path-to-csproj>` with this repo's MTP setup.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [immich-reversegeo/immich-reversegeo](https://github.com/immich-reversegeo/immich-reversegeo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
