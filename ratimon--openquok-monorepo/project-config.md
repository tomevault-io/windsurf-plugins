---
trigger: always_on
description: When public API routes change, keep Swagger JSDoc, SDK, apis-* docs, getting-started overview, and agent CLI (when exposed) in sync in the same change.
---


# Public API surface: keep Swagger, SDK, docs, and CLI in sync

`backend/routes/publicApi/` (mounted at `/api/v1/public/*`) is the **source of truth** for everything an API-key user can call. Any change to a public route — new endpoint, removed endpoint, renamed path, new/changed query/body/response shape — must be reflected in **Swagger, the Node SDK, the docs site, and (when exposed) the agent CLI** in the **same PR**. Do not wait for a follow-up request to add docs.

| Layer | Path | What it provides |
|---|---|---|
| OpenAPI spec | `backend/swagger/jsdoc/*.doc.ts` (+ `openApiSpec.ts` tags) | `/api/v1/openapi.json` + interactive playground |
| Node SDK | `sdk/src/index.ts`, `sdk/src/dtos.ts`, `sdk/README.md` | `@openquok/node-sdk` programmatic client |
| Agent / CLI | `agent/src/api.ts` (+ `commands/`) | Subset of routes exposed as `openquok` commands |
| API reference | `web/src/content/docs/apis-<domain>/*.md` | `/docs/apis-*` pages (body rendered from OpenAPI) |
| Overview | `web/src/content/docs/getting-started-for-public-api/` | Auth, concepts (groups, plugs), SDK quickstart |
| CLI usage | `web/src/content/docs/cli-usages/` | Command recipes when the CLI wraps the route |

**Do not confuse layers:** `backend/swagger/jsdoc/*.doc.ts` files are **OpenAPI JSDoc sources only** (merged into `/api/v1/openapi.json`). They are **not** Vitest specs. Agent tests live under `agent/src/**/*.test.ts` (unit) and `agent/tests/e2e/**/*.e2e.test.ts` (e2e), with e2e filenames like `threads.schedule.post.e2e.test.ts` — see `agent-cli.mdc`.

## 1. Swagger JSDoc (`backend/swagger/`)

For every public route handler, one `@openapi` block must exist in `backend/swagger/jsdoc/`. The frontend renders examples and field tables from this spec, so missing/empty schemas show up as `{"_note": "No JSON example in OpenAPI for this response."}` and empty `ParamField` / `ResponseField` rows.

- **One file per operation**, named `<domain>.<topic>.doc.ts` (e.g. `integrations.public-list.doc.ts`, `integrations.public-plug-catalog.doc.ts`, `posts.public-post-flip-status.doc.ts`). End the file with `export {};` so it stays a module.
- The block must declare:
  - `operationId` (camelCase, stable — SDKs / search rely on it).
  - `tags: [<Tag>]` matching one of the tags registered in `backend/swagger/openApiSpec.ts` (`Integrations`, `Posts`, `Uploads`, …). **Add the tag there first** if you introduce a new section.
  - `security: []` for routes that are **public/unauthenticated**, otherwise omit (the global `ApiKeyAuth` default applies).
  - At least one `responses.<status>.content.application/json.example` (and `requestBody…example` for POST/PUT). The docs UI uses these for the rendered "Response" / "Body" panels.
  - Full `schema` (or `$ref` to a `components.schemas.*` entry) for each request body / response so `flattenSchemaToResponseFields` can produce a complete field table.
- **YAML pitfall**: never start a plain YAML value with a backtick. Quote descriptions that contain code spans: `description: '\`draft\` persists without enqueuing'`.

Reference: `backend/swagger/jsdoc/integrations.public-list.doc.ts`, `backend/swagger/jsdoc/integrations.public-plug-catalog.doc.ts`, `backend/swagger/jsdoc/posts.public-create.doc.ts`.

## 2. SDK (`sdk/`)

`sdk/src/index.ts` is a thin `fetch` wrapper. Each public endpoint should be reachable from a method on the default-exported `Openquok` class.

- Add/rename/remove the method to match the route (same HTTP method, same URL after `${this.apiRoot}`, same header set — `Authorization: <apiKey>` and `Content-Type: application/json` for JSON bodies).
- Put request/response shapes in `sdk/src/dtos.ts` (e.g. `PublicCreatePostDto`, `PublicPlugUpsertBodyDto`). Keep field names identical to the OpenAPI schema so downstream consumers can cast safely.
- Update `sdk/README.md` method table when the surface changes.
- Run `pnpm --filter ./sdk run build`. Bump `sdk/package.json` `version` when publishing.

See also `sdk-maintenance.mdc`.

## 3. Agent CLI (`agent/`)

`agent/src/api.ts` defines `OpenquokApi`, the HTTP client used by the `openquok` CLI. **It is not required to mirror every method on the Node SDK** — only add or change `OpenquokApi` methods for routes that existing or new CLI commands actually call. The SDK (`sdk/src/index.ts`) is the full programmatic client; the CLI is an intentional subset.

- Use the existing `requestJson` helper for JSON endpoints and `form-data` + `node-fetch` for multipart (see `uploadFile`).
- When you add a user-facing CLI verb, add the matching `OpenquokApi` method (reuse the same URL, method, and naming as the SDK method for that operation when one exists) and wire it under `agent/src/commands/`.
- Add or update `web/src/content/docs/cli-usages/<topic>.md` and the `cli-usages/index.md` `CardGrid` when new commands ship.

See also `agent-cli.mdc`.

## 4. Docs (`web/src/content/docs/apis-*/`)

Public API reference pages live under `web/src/content/docs/apis-<domain>/` (e.g. `apis-integrations`, `apis-posts`, `apis-uploads`). Each page is a thin Markdown file whose body is auto-rendered from the OpenAPI spec by `OpenApiDocSplit.svelte`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ratimon/openquok-monorepo](https://github.com/Ratimon/openquok-monorepo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
