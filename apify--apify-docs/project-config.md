---
trigger: always_on
description: Docusaurus-based documentation site combining multiple content types:
---

# Apify Documentation Standards

## Project overview

Docusaurus-based documentation site combining multiple content types:

- **Platform docs** (`sources/platform/`) - Product documentation
- **Academy** (`sources/academy/`) - Educational courses and tutorials
- **API reference** (`apify-api/`) - Generated from OpenAPI specs
- **Multi-repo architecture** - SDK/Client docs from separate repos served via nginx

One unified site built from multiple repositories. See `CONTRIBUTING.md` for multi-repo setup.

## Commands

```bash
pnpm install            # Install dependencies (runs patch-package via postinstall)
pnpm start              # Dev server (rebuilds API docs, port 3000)
pnpm build              # Production build (catches broken links, bad frontmatter)
pnpm lint               # Run all linters (markdownlint + oxlint)
pnpm lint:md            # Markdownlint only
pnpm lint:code          # oxlint only
pnpm lint:fix           # Auto-fix both linters
vale sync               # Download Vale styles (first time only)
vale "path/to/file.md" --minAlertLevel=error  # Prose style check
pnpm api:rebuild        # Regenerate API docs from OpenAPI specs
pnpm openapi:lint       # Validate OpenAPI spec (Redocly + Spectral + YAML)
```

## Architecture

### OpenAPI documentation pipeline

API docs are generated, NOT hand-written. The workflow:

1. **Source of truth**: `apify-api/openapi/openapi.yaml` (splits into `/paths` and `/components`)
1. **Redocly plugins** (`apify-api/plugins/apify.mjs`) inject custom behavior:
   - `code-samples-decorator.mjs` - Auto-adds code samples if files exist in `/code_samples/{js,curl}/`
   - `legacy-doc-url-decorator.mjs` - Adds backward-compatible URLs
   - `client-references-links-decorator.mjs` - Links to client library docs
1. **Build command**: `pnpm api:rebuild` = clean + bundle with Redocly + generate with Docusaurus
1. **Output**: Markdown files in `apify-api/docs/` (gitignored, regenerated on each build)

Never edit generated API docs directly. Edit the OpenAPI YAML source or add code samples.

### OpenAPI code samples

Add code samples by creating files in `apify-api/openapi/code_samples/{javascript,curl}/`:

- Filename must match `operationId` from OpenAPI spec (e.g., `actorRun_get.js`)
- Decorator auto-detects and adds `x-codeSamples` property
- Missing samples are logged during build

### OpenAPI specification changes

- Target OpenAPI specification version should be extracted from `/openapi/openapi.yaml`. All specification changes should be compliant with syntax for that specific OpenAPI specification version.
- Prefer re-use of existing objects via `$ref` over duplication. Reusable components can be found in `/openapi/components`.
- Components most suitable for re-use are:
  - Request parameters and path parameters defined in  `/openapi/components/parameters`
  - Request/response schemas defined in `/openapi/components/schemas`
  - Explicit non-automatic examples defined in `/openapi/components/examples`
  - Objects that are not standardized in the OpenAPI specification defined in `/openapi/components/objects`
- Objects that exist in several variants with only minor differences across different files can be extracted into file in `/openapi/components/objects`. Within one file the YAML anchor syntax can be used to define a shared and unique portion of such objects and avoid some code duplication. These objects will be used only during bundling process through references, but they will not be standalone entities in the generated specification.
- When changing files in `/openapi/paths` look for opportunities to extract shared duplicate objects into re-usable components saved in `/openapi/components`.
- When adding new endpoints, check first if any existing path is similar and if yes, try to re-use same components. If by adding new paths you create new duplication, try to extract it into a new components and reference it instead.
- Prefer automatically generated examples from schema over explicit examples.

#### Error responses

- Re-use schemas for error responses defined in `/apify-api/openapi/components/responses`
- Each endpoint should have at least following error responses: 400 (Bad Request), 405 (Method Not Allowed), 429 (Too Many Requests).
- Endpoints that define `security: []` do not use any authentication.
- Each endpoint that uses authentication should have at least following error responses: 401 (Unauthorized), 403 (Forbidden).
- Each endpoint that has `runs/last` in its path or that has any ID related parameter (for example `actorId`, `buildId`, `runId`, `datasetId` and so on) should have at least one 404 (Not Found) error.
- Each endpoint that has `requestBody` should have at least following error responses: 413 (Payload Too Large), 415 (Unsupported Media Type).

#### Syntax hints
- Instead of using one item `enum`, use `const`:
Avoid this:
```yaml
schema:
  type: string
  enum:
    - "constantValue"
```
Use this:
```yaml
schema:
  type: string
  const: "constantValue"
```

### Python API client model generation

OpenAPI spec changes in this repo automatically trigger Pydantic model regeneration in `apify-client-python`. The pipeline:

1. **This repo** (`.github/workflows/openapi-ci.yaml`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apify/apify-docs](https://github.com/apify/apify-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
