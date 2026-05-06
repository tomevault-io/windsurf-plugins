---
trigger: always_on
description: > Guide for GitHub Copilot Chat when working on `techspokes/typescript-wsdl-client`.
---

# Copilot Instructions for This Repository

> Guide for GitHub Copilot Chat when working on `techspokes/typescript-wsdl-client`.

## First interaction requirement

Before making any changes, read the file-pattern-specific instructions in `.github/instructions/`. These rules apply automatically to matching files and override general conventions. Currently: `markdown.instructions.md` governs all `**/*.md` files.

## Repository Information

- Owner/Repo: techspokes / typescript-wsdl-client
- Package: `@techspokes/typescript-wsdl-client`
- Default Branch: `main`
- CLI Binary: `wsdl-tsc` (`dist/cli.js`)
- Engine: Node.js >= 20
- Issues: https://github.com/techspokes/typescript-wsdl-client/issues
- Discussions: https://github.com/techspokes/typescript-wsdl-client/discussions
- CI Workflow: `.github/workflows/ci.yml`
- Version Policy: always read `"version"` from `package.json`; do not duplicate it here.

Machine-readable metadata (parse first for GitHub MCP operations):

```yaml
# repo-metadata
owner: techspokes
repo: typescript-wsdl-client
package: '@techspokes/typescript-wsdl-client'
# version: always read from package.json
defaultBranch: main
issues: 'https://github.com/techspokes/typescript-wsdl-client/issues'
discussions: 'https://github.com/techspokes/typescript-wsdl-client/discussions'
ciWorkflow: 'ci.yml'
cliBinary: wsdl-tsc
node: '>=20'
```

### GitHub MCP Usage Guidelines (minimal)

1. Use `owner` / `repo` from YAML; read `package.json` each time you need the version (no caching).
2. Commit/PR titles: prefix with `Version: <version>` (see commit format rules).
3. Default PR base: `main` unless explicitly overridden.
4. Prefer repo-scoped queries; avoid broad GitHub-wide searches for issues/PRs.
5. For releases, read `CHANGELOG.md` and `package.json` and follow the release workflow below.
6. Don't remote-fetch metadata already present in this file or `package.json`.
7. For any code or docs change, propose (and on confirmation add) a concise bullet under `## [Unreleased]` in `CHANGELOG.md` (omit the `Version: <'>` prefix).

## Project Context

Purpose: generate fully-typed TypeScript SOAP clients from WSDL/XSD, with optional OpenAPI 3.1 and Fastify REST gateway.

Key features: deterministic JSON/SOAP metadata with attribute and element flattening; `$value` convention for text content; inheritance resolution for complex and simple content; choice handling strategies; WS-Policy hints; string-first primitive mapping by default; catalog introspection via `catalog.json` reused across all generation stages.

Primary users: TypeScript developers integrating with SOAP services.

Maintainer: Serge Liatko ([@sergeliatko](https://github.com/sergeliatko)). Vendor: TechSpokes (https://www.techspokes.com).

### Runtime and tooling

- Node.js `>= 20.0.0` (see `engines.node` in `package.json`).
- TypeScript strict, ES2022, `type: "module"` (ESM/NodeNext style).
- CLI binary: `wsdl-tsc` (entry: `dist/cli.js`), orchestrating the pipeline.
- The `soap` package is a runtime dependency for consumers. The `wsdl-tsc` CLI itself is a devDependency in consumer projects.

### Generated artifacts

- Client: `client.ts`, `types.ts`, `utils.ts`, `operations.ts`, co-located `catalog.json`.
- OpenAPI: 3.1 spec (`.json`/`.yaml`) mirroring the TS model.
- Gateway: Fastify route handlers with SOAP client calls, envelope wrapping, error handling, and runtime array unwrap (`plugin.ts`, `routes.ts`, `routes/`, `schemas.ts`, `schemas/`, `runtime.ts`, `_typecheck.ts`).

### Catalog co-location

Important for reasoning about paths:
- `compile`: explicit `--catalog-file` (no default).
- `client`: defaults to `{client-dir}/catalog.json`.
- `openapi`: defaults to `{dir-of-openapi-file}/catalog.json`.
- `pipeline`: cascade lookup: `{client-dir}` / `{openapi-dir}` / `{gateway-dir}` / `tmp/`.

Scratchpads: non-project notes may live under ad-hoc folders; treat them as scratchpads only, not as source of truth for behavior or docs.

### Project evolution

The project evolved from a simple WSDL parser (v0.1.x) to a full generation pipeline (v0.9.x through v0.10.x). The gateway generator reads OpenAPI specs, not the catalog directly, to produce Fastify route handlers. Since v0.10.0 the gateway has end-to-end type safety from WSDL definitions through to HTTP response types. Since v0.11.0 the project includes a full Vitest test suite, a typed operations interface for mocking, and runtime ArrayOf* unwrap.

## Project Structure

Source is in `src/` with these modules: `loader/` (WSDL fetching and parsing), `compiler/` (XSD to catalog compilation), `client/` (TypeScript client and operations interface generation), `openapi/` (OpenAPI 3.1 spec generation), `gateway/` (Fastify route handler and runtime generation), `app/` (standalone application scaffold), `util/` (shared helpers, CLI builder, errors), `xsd/` (XSD type definitions), and `types/` (shared TypeScript type definitions).

The pipeline orchestrator is `src/pipeline.ts`. Configuration defaults are in `src/config.ts`.

## CLI and Commands (align with README)

CLI entry: `wsdl-tsc` (via `npx wsdl-tsc` in README examples).

Commands (6 total; keep terminology consistent with README):
- `compile`: WSDL / `catalog.json` only (no TS code).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TechSpokes/typescript-wsdl-client](https://github.com/TechSpokes/typescript-wsdl-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
