---
trigger: always_on
description: handles persistence in `save()`. Reads of those collections go through
---

# ActivityPub AI Assistant Guide

This file provides comprehensive guidance for AI agents contributing to this repository.

> **Note:** This document contains detailed code examples and implementation patterns. For a concise human-readable overview, see [README.md](README.md).

## Project Overview

A multitenant ActivityPub server for [Ghost](https://ghost.org/), built with
[Fedify](https://fedify.dev/). This service makes it possible for independent
websites to publish their content directly to the Fediverse, enabling networked
publishing to the open social web

## Tools & Technologies Used

- [Node.js](https://nodejs.org) - Runtime
- [TypeScript](https://www.typescriptlang.org) - Programming language
- [Yarn](https://yarnpkg.com) - Node package management
- [Biome](https://biomejs.dev) - Linter & code formatter
- [esbuild](https://esbuild.github.io/) - Bundler
- [Hono](https://hono.dev) - Web Server
- [Fedify](https://fedify.dev) - Federation
- [Awilix](https://github.com/jeffijoe/awilix) - Dependency injection
- [Zod](https://zod.dev) - Schema validation
- [Knex](https://knexjs.org/) - SQL query builder
- [Vitest](https://vitest.dev) - Testing (unit / integration)
- [Cucumber](https://cucumber.io) - Testing (e2e)
- [Wiremock](https://wiremock.org) - API mocking (for e2e tests)
- [migrate](https://github.com/golang-migrate/migrate) - Database migrations
- [Docker](https://www.docker.com) - Containerisation
- [Docker Compose](https://docs.docker.com/compose) - Container orchestration
- [MySQL](https://www.mysql.com) - Database
- [Google Cloud Cloud Run](https://cloud.google.com/run) - Production deployment
- [Google Cloud Cloud SQL](https://cloud.google.com/sql) - Production database deployment
- [Google Cloud Pub/Sub](https://cloud.google.com/pubsub) - Production messaging
- [Google Cloud Cloud Storage](https://cloud.google.com/storage) - Production file storage

---

## Repository Structure

- `/dev` - Development related tools, configurations, and utilities
- `/features` - Cucumber feature files for e2e testing
- `/jobs` - One-off jobs to be executed in a production environment (Google Cloud)
- `/migrate` - Database migrations
- `/src` - Source code for the application

---

## Code Quality

### Linting & formatting

To run the linter:

```bash
yarn lint
```

To run the formatter:

```bash
yarn fmt
```

### Type checking

To run type checking:

```bash
yarn test:types
```

### Testing

To run all tests (slow):

```bash
yarn test
```

To run unit tests only (fast):

```bash
yarn test:unit
```

To run integration tests only (slow):

```bash
yarn test:integration
```

To run e2e tests only (slow):

```bash
yarn test:cucumber
```

To run a single unit / integration test (fast):

```bash
yarn test:single 'path/to/test'
```

To run a single e2e test (slow):

1. Add a `@only` tag either above a feature file OR a scenario in a feature file:

```cucumber
# hello-world.feature

@only
Feature: Hello world

    Scenario: It prints "Hello, world!"
        ...
```

2. Run the test:

```bash
yarn test:cucumber @only
```

#### Testing Guidelines:

- Cover as much as possible with unit tests
- Use integration tests for anything that cannot be reasonably unit tested
- Use e2e tests to cover features at a high level
- All unit & integration test files should have the prefix `.test.ts`
- The type of test should be indicated by the file extension:
  - `.unit.test.ts` for unit tests
  - `.integration.test.ts` for integration tests
- Tests should be co-located with the code they test
- e2e tests should reside in the `features` directory
- Tests should execute quickly, there is an upper limit of 10 seconds
- Assertions should target the specific field being tested, not stringify
  the whole object. Prefer `expect(result.field).toEqual(...)` over
  `expect(JSON.stringify(result)).toContain(...)` — the latter passes when
  the value appears in any field, hiding bugs
- When adding a new table, also add it to `FixtureManager.reset()` so
  integration tests stay isolated between runs

#### Testing Notes:

- Tests are executed within a Docker container when executed via `yarn`. This
  means extra flags passed to `yarn` will not be passed to the test runner

---

## Development Environment

### Setup

#### Using Tailscale

Use [Tailscale](https://tailscale.com) to expose the local machine to the internet:

```bash
tailscale funnel 80
```

### Services

- [Nginx](https://nginx.org) - Reverse proxy used to proxy traffic from port
  `80` to port `8080` if traffic is meant for activitypub, or forward on to the
  docker host (`host.docker.internal`) for any other traffic (i.e Ghost)
- [MySQL](https://www.mysql.com) - Database
  - Port: `3307`
  - User: `ghost`
  - Password: `password`
  - Database: `activitypub`
- [Google Cloud Pub/Sub emulator](https://cloud.google.com/pubsub/docs/emulator) - Pub/Sub emulator
  - Port: `8085`
- [Google Cloud Storage emulator](https://github.com/fsouza/fake-gcs-server) - Storage emulator
  - Port: `4443`

### Run the application

```bash
yarn dev
```

### Run the application with logging to the console

```bash
yarn dev && yarn logs
```

### Stop the application

```bash
yarn stop
```

This will also stop any service dependencies

### Wipe the database

```bash
yarn wipe-db
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TryGhost/ActivityPub](https://github.com/TryGhost/ActivityPub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
