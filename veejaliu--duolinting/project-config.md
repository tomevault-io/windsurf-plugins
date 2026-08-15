---
trigger: always_on
description: This file is the entry point for AI coding agents. It assumes you know nothing
---

# DuolinTing Agent Guide

This file is the entry point for AI coding agents. It assumes you know nothing
about the project.

## Project Overview

DuolinTing is a front-end/back-end separated intensive-listening product.
It is an npm-workspaces monorepo named `duolinting`, with four deployable
applications and five shared packages:

- `backend` (`@duolinting/backend`): unified Node.js backend for catalog,
  exercises, auth, learner progress, feedback, admin content operations, and
  media upload intents. Express 5 + Sequelize (mysql2) + MinIO client, written
  in TypeScript, compiled to CommonJS with `tsc`, run in dev with `tsx watch`.
- `web-app` (`@duolinting/web-app`): learner-facing listening app. React 19 +
  Vite + Radix UI + react-router-dom, port 8101 in dev.
- `admin` (`@duolinting/admin`): administrator content-management app. Same
  stack as web-app plus wavesurfer.js, port 8102 in dev. Ant Design is the
  workspace UI framework (with Radix Select/Dialog inside the course-production
  workbench). Keep it professional and operational, not playful.
- `mobile-app` (`@duolinting/mobile-app`): Expo (~54) app using expo-router,
  nativewind (Tailwind), @tanstack/react-query, and zustand. Routes live in
  `mobile-app/app/`; feature code in `mobile-app/src/`. Its own `AGENTS.md`
  says: read the versioned Expo docs at https://docs.expo.dev/versions/v54.0.0/
  before writing any code — Expo APIs have changed.
- `packages/domain`: shared domain contracts (auth, catalog, exercises,
  progress types) used by all clients.
- `packages/shared`: shared domain contracts and system-level constants for
  backend/admin.
- `packages/api-client`: typed fetch-based API client (`ApiClientError`,
  request helpers) built on `@duolinting/domain` + `@duolinting/app-config`.
- `packages/app-config`: runtime config — API base URL resolution
  (`VITE_API_BASE_URL` / `EXPO_PUBLIC_API_BASE_URL`), auth token storage keys,
  feature flags.
- `packages/ui-tokens`: shared design tokens.

Infrastructure (local, `docker-compose.yml`): MySQL 8.4 (host port 3307),
MinIO (API 9000, console 9001), and a one-shot Flyway service (profile `tools`)
that runs the migrations in `infra/mysql/migrations`.

Local dev services:

- Learner app: `http://127.0.0.1:8101`
- Admin app: `http://127.0.0.1:8102`
- Backend health: `http://127.0.0.1:8100/api/health`
- Backend inspector (dev:debug): `127.0.0.1:9229`
- MinIO console: `http://127.0.0.1:9001` (default local creds `minioadmin` / `minioadmin`, bucket `duolinting-media`)
- Default local database: `duolinting_app_dev`

## Agent Working Rules (from the project owner)

- When using the owner's signed-in browser for a multi-step workflow, keep pages open whenever the workflow is awaiting owner confirmation or may need to continue. Preserve the current page and entered state; close a tab only after the workflow is complete, the owner explicitly asks to close it, or the page is demonstrably unusable.
- To save time, do not ask the owner to confirm source snippets, run routine Git
  status checks, take screenshots after deployment, or open a browser to inspect
  deployed pages. The owner verifies pages and functionality manually. Perform
  these checks only when the current conversation explicitly requests them.
  Before a commit, release, or open-source publication, one read-only review is
  allowed to confirm the change scope and detect accidentally included secrets,
  production addresses, private configuration, or large files. Do not use that
  review as an opportunity to modify or clean up unrelated changes.
- Add clear comments around critical logic such as calculations, conversions,
  validation, and custom data formats. Explain each field's format and meaning,
  as well as why the implementation handles it that way.
- On learner clients (`web-app` and `mobile-app`), prefer Radix UI or React Native
  components appropriate to the platform and follow the existing design system.
  Ant Design is the primary UI framework for Admin; the existing Radix
  Select/Dialog components in the course-production workbench may remain. Reuse
  existing dependencies and interaction patterns before introducing new ones.
- Use `#1cb0f6` (Duolingo blue) for hover-border changes across buttons, tags,
  cards, and other interactive elements. A normal, unselected element must use
  this color when its border changes on hover. Active or success states may use
  a darker shade of their own semantic color where appropriate.

## Build and Run Commands

Root scripts (npm workspaces; packages build in dependency order
domain → app-config → ui-tokens → api-client → shared):

```bash
npm install                 # install all workspaces
npm run infra:up            # start MySQL + MinIO via docker compose
npm run db:migrate          # run Flyway migrations (docker compose tools profile)
npm run dev                 # build packages, then run backend + web-app + admin
npm run dev:debug           # same, with backend inspector on 9229
npm run dev:backend         # backend only (tsx watch)
npm run dev:web-app         # web-app only (vite, 127.0.0.1:8101)
npm run dev:admin           # admin only (vite, 127.0.0.1:8102)
npm run dev:mobile-app      # build packages, then expo start

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VeejaLiu/duolinting](https://github.com/VeejaLiu/duolinting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-15 -->
