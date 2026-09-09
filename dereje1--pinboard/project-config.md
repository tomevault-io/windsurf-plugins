---
trigger: always_on
description: > **Purpose:** A predictable place for coding agents (Codex, Cursor, Factory, Aider, Gemini CLI, etc.) to learn how to build, test, and contribute to this repository without unnecessary back-and-forth.
---

# AGENTS.md — Pinterest-Clone

> **Purpose:** A predictable place for coding agents (Codex, Cursor, Factory, Aider, Gemini CLI, etc.) to learn how to build, test, and contribute to this repository without unnecessary back-and-forth.

## 0) Operating context

- Repository: `Dereje1/Pinterest-Clone`
- Default branch: `master`
- This is an existing application being restored and improved, not a greenfield rewrite.
- The GitHub repository and current implementation are the source of truth.
- The README and older documentation may be stale. Verify behavior against the code before making assumptions.
- Follow the scope of the assigned GitHub Issue exactly.
- Avoid sweeping refactors, dependency upgrades, architecture changes, or unrelated cleanup unless the Issue explicitly asks for them.
- If an Issue conflicts with this file, stop and surface the conflict rather than silently inventing behavior.

The current restoration priority is:

1. Keep the existing application working.
2. Fix confirmed build/runtime problems with the smallest safe change.
3. Preserve existing data and integrations.
4. Add targeted verification for repaired behavior.
5. Modernize dependencies only in separately scoped work.

---

## 1) Project overview

This is a full-stack Pinterest-style TypeScript application.

### Frontend

- React 17
- React Router 5
- Redux / Redux Toolkit
- Material UI
- Vite

### Backend

- Express
- TypeScript
- MongoDB through Mongoose
- Cookie/session-based authentication infrastructure
- OAuth integrations

### External services and integrations

- MongoDB
- AWS S3 image storage
- Google authentication
- GitHub authentication
- Twitter/X authentication
- OpenAI image generation

### User-facing capabilities

The application includes functionality for:

- browsing existing pins
- image uploads
- creating pins from URLs
- creating pins as an authenticated user
- OpenAI image generation
- authentication through supported OAuth providers

Treat these existing flows as behavior to preserve unless an Issue explicitly changes them.

---

## 2) Repository structure and source of truth

Before changing code:

1. Inspect the relevant implementation.
2. Trace the current code path.
3. Identify the smallest set of files required.
4. Check whether the change touches authentication, MongoDB, S3, deployment, routing, environment variables, or external APIs.
5. Preserve current behavior outside the Issue scope.

Do not rely on the README alone when determining how the application currently works.

Do not perform broad search-and-replace operations across the repository without reviewing every changed occurrence.

---

## 3) Dev environment and running locally

This repository uses **npm**.

### Install dependencies

Use the repository's existing lockfile and package-manager conventions.

```bash
npm install
```

If dependency installation fails because of legacy peer-dependency conflicts, inspect the failure before choosing a workaround. Do not casually regenerate or replace the lockfile during an unrelated Issue.

### Run the full development environment

```bash
npm run dev
```

This runs the frontend and backend concurrently.

Equivalent individual commands:

```bash
npm run client
npm run server
```

- `npm run client` starts the Vite client.
- `npm run server` starts the backend through Nodemon.

Local execution may require environment variables and access to external services such as MongoDB, AWS S3, OAuth providers, or OpenAI.

Never invent production credentials.

If an external integration cannot be exercised locally, preserve the existing integration and clearly report the limitation.

---

## 4) Build, type-check, lint, and test

The current repository scripts are:

### TypeScript verification

```bash
npm run compileTS
```

This verifies:

- client TypeScript
- server TypeScript with `--noEmit`
- test TypeScript

### Lint

```bash
npm run lint
```

This runs ESLint across:

- `server`
- `client`
- `tests`

### Client production build

```bash
npm run build_client
```

This compiles the client TypeScript and runs the Vite production build.

### Server production build

```bash
npm run build_server
```

This compiles the server and `bin` TypeScript projects.

### Production start

After the server build:

```bash
npm run start
```

This starts:

```text
node ./server_build/bin/www
```

### Tests

The test runner is **Jest**.

The repository's default command is:

```bash
npm run test
```

However, the package script runs Jest in watch mode. Do not use watch mode as the final PR verification command.

For a one-shot full test run, use:

```bash
npm test -- --watch=false
```

If serial execution is needed for stability:

```bash
npm test -- --watch=false --runInBand
```

Coverage:

```bash
npm run coverage
```

### Narrowing a failing Jest test

Examples:

```bash
npx jest path/to/test-file.test.ts --runInBand
```

or:

```bash
npx jest -t "<test name or regex>" --runInBand
```

Use targeted tests during development, but run the required full checks before opening a PR unless the Issue or environment makes that impossible.

---

## 5) Required pre-PR verification


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dereje1/Pinboard](https://github.com/Dereje1/Pinboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
