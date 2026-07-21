---
trigger: always_on
description: Meshery is a self-service engineering platform and open source cloud native manager for Kubernetes infrastructure. A CNCF project supporting 300+ integrations with visual GitOps, multi-cluster management, and workspace collaboration.
---

# Project Overview

Meshery is a self-service engineering platform and open source cloud native manager for Kubernetes infrastructure. A CNCF project supporting 300+ integrations with visual GitOps, multi-cluster management, and workspace collaboration.

## Repository Structure

| Directory | Purpose |
|-----------|---------|
| `/server` | Go backend — REST/GraphQL APIs, Kubernetes management, PostgreSQL |
| `/ui` | Next.js/React frontend — MUI, Redux Toolkit, Relay GraphQL |
| `/mesheryctl` | Go CLI with Cobra — install, lifecycle, pattern deployment |
| `/docs` | Hugo documentation site |
| `/install` | Dockerfiles, Kubernetes manifests, Helm charts |
| `/provider-ui` | Provider-specific React UI extensions |
| `/.github` | GitHub Actions, issue templates, Copilot agent definitions |

## Identifier Naming Conventions — MANDATORY

Authoritative guide: <https://github.com/meshery/schemas/blob/master/docs/identifier-naming-contributor-guide.md>

**Wire is camelCase; DB is snake_case; Go fields follow Go idiom; the ORM layer is the sole translation boundary.**

### Per-layer canonical forms

| Layer | Form |
|---|---|
| DB column / `db:` tag | `snake_case` — `user_id`, `org_id`, `created_at` |
| Go struct field | `PascalCase` with Go initialisms — `UserID`, `OrgID`, `CreatedAt` |
| JSON tag | `camelCase` — `json:"userId"`, `json:"orgId"` |
| URL query/path param | `camelCase` — `{orgId}`, `?userId=...` |
| TypeScript property | `camelCase` — `response.userId` |
| OpenAPI schema property | `camelCase` |
| OpenAPI `operationId` | `lower camelCase verbNoun` — `getWorkspaces` |
| `components/schemas` type name | `PascalCase` — `WorkspacePayload` |

### Forbidden (MUST NOT)

- MUST NOT use a `json:` tag matching the `db:` tag — wire is camel, DB is snake.
- MUST NOT hand-roll an RTK query endpoint when `@meshery/schemas/{mesheryApi,cloudApi}` provides one.
- MUST NOT locally redeclare a Go type with an equivalent in `github.com/meshery/schemas/models/...`.
- MUST NOT use `ID` (ALL CAPS) in URL params, JSON tags, or TypeScript properties — use `Id`.
- MUST NOT mix casing within a single resource; introduce a new API version to change wire format.
- MUST NOT import deprecated `v1beta1` in new code; use `v1beta3` (or `v1beta2` where v1beta3 absent).

### Required on every PR

- Run schemas validator: `cd ../schemas && make validate-schemas && make consumer-audit`
- Include test updates for casing/tag changes.
- Include doc updates for user-visible API changes.
- Sign off commits: `git commit -s`

> `meshery/schemas/AGENTS.md` is authoritative. On conflicts, schemas wins.

## API Changes — MUST Go Through Schemas — MANDATORY

**Any new or changed HTTP API (new endpoint, new/renamed query param, new
request/response field) MUST be defined in `meshery/schemas` first and consumed
via the generated client. Do NOT hand-roll RTK Query endpoints, response types,
or ad-hoc `fetch`/`axios` calls for an API that can live in schemas.**

Schemas is the single source of truth: one OpenAPI definition drives the Go
models, TypeScript types, and the RTK Query client (`@meshery/schemas/{mesheryApi,cloudApi}`)
consumed here. Hand-rolling any of these silently diverges the wire contract
across `meshery/meshery` and `meshery-cloud`.

### Workflow (adding/updating an endpoint)

1. **Define** the path + schemas in the matching construct's `api.yml`
   (e.g. `../schemas/schemas/constructs/v1beta1/system/api.yml` for `/api/system/*`,
   `.../connection/api.yml` for `/api/integrations/connections*`). Follow the
   schemas conventions: `operationId` = lower-camel `verbNoun`, camelCase wire
   params/properties, `x-internal: ["meshery"]` for Meshery-only endpoints,
   `additionalProperties: false`, `maxLength` on strings.
2. **Regenerate** in `../schemas`: `make bundle-openapi generate-rtk generate-golang`
   (or `make build` for the full dist). Verify the new `useXQuery` /
   `mesheryApi.endpoints.X` hooks appear.
3. **Validate**: `cd ../schemas && make validate-schemas && make consumer-audit`.
4. **Consume** the generated hook in the UI (import from `@meshery/schemas/mesheryApi`;
   wrap in `ui/rtk-query/*` only for thin ergonomics like bare-id args or cache
   tags — never to re-declare the request). Use the generated Go models on the
   server where applicable.
5. **Release coupling**: schemas releases are automated ("do not manually create
   releases"). Until a new `@meshery/schemas` is published and this repo's
   dependency is bumped, a **local link** is used for development
   (`ui/package.json` → `"@meshery/schemas": "file:../schemas"` and the
   `replace github.com/meshery/schemas => ../schemas` directive in `go.mod`).
   Both the version bump and reverting the local link happen as part of the
   normal release/upgrade flow — do not commit the local link as the permanent
   dependency.

### Narrow exceptions (still prefer schemas)

- **Server-Sent Events / streaming**: RTK codegen can't produce a useful hook
  for `text/event-stream`. Still **document** the endpoint in `api.yml`, but
  consume it with a native `EventSource` client under `ui/lib/*`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meshery/meshery](https://github.com/meshery/meshery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
