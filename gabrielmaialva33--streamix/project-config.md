---
trigger: always_on
description: Central instructions for coding agents working on **Streamix**. `CLAUDE.md` and `GEMINI.md` import this file with
---

# AGENTS.md

Central instructions for coding agents working on **Streamix**. `CLAUDE.md` and `GEMINI.md` import this file with
`@AGENTS.md`, so update this document only.

> Principle: this file only captures repo-specific rules. When a rule is not listed here, follow standard
> Elixir / Phoenix 1.8 / LiveView 1.2 conventions and the detailed guidance in
> [`docs/phoenix-guidelines.md`](docs/phoenix-guidelines.md).

## Project Overview

Streamix is a Phoenix + LiveView streaming platform that aggregates multiple Xtream Codes providers into a single web
application and REST API. The current tree includes:

- password-based authentication and role-based access
- personal providers plus optional system-wide global provider
- optional GIndex ingestion for Google Drive-backed catalogs
- live channels, movies, series, episodes, favorites, history, and watch progress
- AI-assisted semantic search and recommendations when embeddings + Qdrant are configured
- watch parties with synchronized playback, presence, and room chat
- subscription plans, premium access gates, and an admin panel
- signed stream URLs, a Phoenix stream proxy, and a live stream multiplexer
- PWA assets, offline metadata sync hooks, and a mobile / TV-facing REST API

The application release version is sourced from the root `VERSION` file (currently `0.0.100`) and consumed by
`mix.exs`. Historical release tags go through `v1.5.0`; do not infer feature availability by comparing those two
version lines. Prefer the current tree and git history.

This repository is the Phoenix backend + web UI. The older in-repo TV app was extracted to a separate repository, so
do not document or modify an in-tree Tizen app here.

## Stack

- Elixir `~> 1.20`
- OTP 29
- Phoenix `~> 1.8.2`
- Phoenix LiveView `~> 1.2.0`
- Ecto SQL `~> 3.14`
- TimescaleDB on PostgreSQL 17 with `pg_trgm`
- Redis 8
- Qdrant (optional, for semantic search)
- RabbitMQ 4 + Broadway (optional)
- Oban 2.23
- Req + Finch
- Tailwind CSS v4
- esbuild
- npm 12-managed frontend packages in `assets/`

## Local Setup

```bash
docker compose up -d
cp .env.example .env
cd assets && npm ci && cd ..
mix setup
mix phx.server
```

Important local setup facts:

- `mix setup` runs `ecto.setup`, seeds, and asset builds.
- `priv/repo/seeds.exs` requires `ADMIN_PASSWORD`.
- provider credential encryption requires `PROVIDER_ENCRYPTION_KEY`.
- `assets/node_modules` is ignored, so local JS dependencies require `npm ci`.
- if `TEST_DATABASE_URL` is not set, test config derives it from `DATABASE_URL`.

## Common Commands

| Command                        | Purpose                                                      |
|--------------------------------|--------------------------------------------------------------|
| `mix setup`                    | deps, DB setup, seeds, asset setup/build                     |
| `mix phx.server`               | run the web app                                              |
| `iex -S mix phx.server`        | run with IEx                                                 |
| `mix test`                     | full test suite                                              |
| `mix test path/to/test.exs`    | targeted test file                                           |
| `mix test path/to/test.exs:42` | targeted test by line                                        |
| `mix precommit`                | compile warnings-as-errors, deps.unlock, format, credo, test |
| `mix quality`                  | compile, credo, test, dialyzer                               |
| `mix ecto.migrate`             | run migrations                                               |
| `mix ecto.reset`               | drop, create, migrate, seed                                  |
| `mix assets.build`             | build CSS + JS                                               |
| `mix assets.deploy`            | minify and digest assets                                     |
| `cd assets && npm ci`          | install frontend dependencies                                |

## Dependency Currency

- Track the latest stable releases for Hex, npm, Docker / Compose, and GitHub Actions. Dependabot checks all four
  ecosystems daily through `.github/dependabot.yml`.
- Keep GitHub Actions pinned to immutable commit SHAs and retain the release tag in a comment.
- A stateful infrastructure major (PostgreSQL, TimescaleDB, Redis, RabbitMQ, Qdrant) still requires a rehearsed data
  migration and rollback; "latest" never means replacing a production data format blindly.

## Project Structure

```text
lib/streamix/
├── access/                   # Permissions, role_permissions, user_permissions
├── accounts/                 # Users, roles, auth tokens, IP tracking
├── ai/                       # Embeddings, Gemini, NVIDIA, Qdrant, recommendations
├── billing/                  # Plans, subscriptions, premium access rules
├── cache/                    # L1 ConCache and L2 Redis implementations
├── ecto/                     # Shared Ecto types
├── gindex/                   # GIndex transport, parsing, matching, quota, and sync
├── iptv/                     # Canonical catalog, providers, EPG, engagement, and streaming

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gabrielmaialva33/streamix](https://github.com/gabrielmaialva33/streamix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
