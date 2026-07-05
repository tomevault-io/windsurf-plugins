---
trigger: always_on
description: Welcome to the Manifold project! This file provides essential context, setup instructions, and architectural rules that any AI agent or developer must strictly follow when working on this codebase.
---

# Manifold - AI Developer Guidelines

Welcome to the Manifold project! This file provides essential context, setup instructions, and architectural rules that any AI agent or developer must strictly follow when working on this codebase.

## 1. Tech Stack & Project Overview

Manifold is a game storefront and catalog application. It is crucial to understand the tools and routing paradigm used:

- **Framework:** Next.js using the **Pages Router** (`pages/api/...`), NOT the App Router.
- **Language:** Strictly **TypeScript**. The use of `any` is expressly forbidden. Always define proper interfaces and types.
- **Database:** PostgreSQL.
- **Styling:** Tailwind CSS.

### Directory Structure

- `pages/api/` -> Controllers / API Route Handlers.
- `models/` -> Core business logic, database queries, and Zod schemas.
- `infra/` -> Core infrastructure configurations (webserver, database connections) and custom error classes.
- `tests/` -> Automated tests. The test orchestrator (`orchestrator.js`) is located at the root of this folder.

## 2. How to Run Locally

1. **Prerequisites:**
   - Ensure your Node.js version matches the one in `.nvmrc` (`v24.13.1`).
   - Docker **must** be installed and running on your system.
2. **Install dependencies:**
   ```bash
   npm i
   ```
3. **Start the development server:**
   ```bash
   npm run dev
   ```
   > **Note:** `npm run dev` automatically handles all necessary environment setups and calls, so you don't need to manually configure `.env` variables or start external services before running it.

## 3. Architecture & Development Guidelines

### Dependency Management

- **Use Exact Versions:** When installing new packages, always use the `-E` (or `--save-exact`) flag to ensure exact versions are pinned in `package.json` (e.g., `npm install -E <package>`). This prevents unexpected breaking changes from minor/patch updates.

### Test-Driven Development (TDD)

### Final Verification

- **MANDATORY:** Before considering a task finished, you must run ALL tests (not just the ones related to your change) to ensure no regressions were introduced. A task is only complete when the entire suite passes.

### Running Tests

- **From scratch:** `npm run test`
- **Watch mode:** To test continuously, you must run `npm run dev` in one terminal, and then run `npm run test:watch` in parallel in another terminal.

### Integration Tests

Focus on writing robust **integration tests**. Use `tests/integration/api/v1/users/post.test.ts` as the primary reference example.

- **One file per method:** Each HTTP method must have its own dedicated test file (e.g., `get.test.ts`, `post.test.ts`, `delete.test.ts`, `patch.test.ts`). Do not group multiple methods into a single `index.test.ts` file.
- Group tests by user states (e.g., `describe("Anonymous user")`, `describe("Authenticated user")`).
- Validate exact response codes (`201`, `400`, `403`) and precise JSON payloads. When testing errors, follow this exact assertion pattern:

  ```typescript
  expect(response.status).toBe(401);

  const responseBody = await response.json();
  expect(responseBody).toEqual({
    message: "Invalid credentials",
    name: "UnauthorizedError",
    action: "Check your credentials",
    status_code: 401,
  });
  ```

### The Test Orchestrator

The `tests/orchestrator.js` file is the core utility for test environment setup.

- **Purpose:** It manages database states, service readiness, and mock data generation.
- **Usage:** Always use it in `beforeAll` to `waitForAllServices()` and `clearDatabaseRows()`. Use its helper methods (`createUser`, `createSession`, etc.) to securely set up test states without needing to make HTTP calls to your own API.

### Database Constraints (CRITICAL)

- **No Foreign Keys:** The database architecture strictly forbids the use of Foreign Keys. Do not create FK constraints in migrations or schemas to ensure maximum horizontal scalability.

### MVC Architecture

The system uses a Model-View-Controller architecture built on Next.js API routes, leveraging `next-connect`.

- **Controllers / API Handlers:** Found in `pages/api/...`.
- **Router Pattern:** Always use named handler functions and pass `controller.canRequest` as a middleware directly in the method call. Avoid anonymous arrow functions in the router chain.

  ```typescript
  export default createRouter<NextApiRequest, NextApiResponse>()
    .use(controller.injectAnonymousOrUser)
    .get(getHandler)
    .post(controller.canRequest("feature:name"), postHandler)
    .handler(controller.errorHandlers);

  async function getHandler(req, res) { ... }
  async function postHandler(req, res) { ... }
  ```

- **Models:** Found in `models/`. They encapsulate database queries, business logic, schemas, and structural integrity.

### Error Handling Protocol

- Do not return generic error responses like `res.status(400).json({ error: '...' })`.
- **Always** `throw` custom error classes from `infra/errors` (e.g., `throw new ValidationError(...)`, `ForbiddenError`, `NotFoundError`).
- These thrown errors are automatically caught and formatted by the `controller.errorHandlers` middleware.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pedromello/manifoldpowered.com](https://github.com/pedromello/manifoldpowered.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
