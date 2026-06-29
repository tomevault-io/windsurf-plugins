---
trigger: always_on
description: This file provides context for AI agents working in this repository.
---

# AGENTS.md — Backstage OPA Plugins

This file provides context for AI agents working in this repository.

## What this repo is

A Yarn monorepo containing a collection of published Backstage plugins and modules that integrate Open Policy Agent (OPA) with Backstage. The goal is to decouple authorization policy from application code — policies live in `.rego` files, are evaluated by a running OPA server, and can be updated without redeploying Backstage.

Each plugin has a `dev/` folder with a self-contained dev backend or frontend harness — no shared core app is needed.

## Repo structure

```
plugins/          — published plugins and modules (see below)
policies/         — example Rego policies for local OPA development
opa-docs/         — Docusaurus documentation site
.claude/skills/   — Claude Code skills for working with this repo
```

## Plugins at a glance

### Published plugins

| Directory                                               | npm package                                                 | Type                                            | Purpose                                                                                                                                                                                                                   |
| ------------------------------------------------------- | ----------------------------------------------------------- | ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `plugins/permission-backend-module-opa-wrapper`         | `@parsifal-m/plugin-permission-backend-module-opa-wrapper`  | Backend module                                  | Wraps the Backstage Permission Framework — delegates permission decisions to OPA. Self-registers, no policy code needed in TypeScript.                                                                                    |
| `plugins/backstage-opa-backend`                         | `@parsifal-m/plugin-opa-backend`                            | Backend plugin                                  | Provides HTTP routes used by `opa-authz-react` and `opa-entity-checker` to evaluate policies. Required for those frontend plugins.                                                                                        |
| `plugins/opa-node`                                      | `@parsifal-m/backstage-plugin-opa-node`                     | Backend library                                 | Provides `opaService` — a Backstage service ref that any backend plugin can inject to call OPA directly for route-level authorization.                                                                                    |
| `plugins/backstage-plugin-opa-authz-react`              | `@parsifal-m/backstage-plugin-opa-authz-react`              | Web library (`web-library`)                     | React components (`RequireOpaAuthz`) and hooks (`useOpaAuthz`, `useOpaAuthzManual`) for hiding/showing UI elements based on OPA decisions. **Legacy frontend system only — not yet migrated to the new frontend system.** |
| `plugins/backstage-opa-entity-checker`                  | `@parsifal-m/plugin-opa-entity-checker`                     | Frontend plugin                                 | Entity page card that shows whether an entity passes an OPA validation policy.                                                                                                                                            |
| `plugins/backstage-plugin-opa-entity-checker-processor` | `@parsifal-m/backstage-plugin-opa-entity-checker-processor` | Backend plugin module (`backend-plugin-module`) | Catalog processor that validates entity metadata during ingestion using OPA and adds annotation results.                                                                                                                  |
| `plugins/backstage-opa-policies`                        | `@parsifal-m/plugin-opa-policies`                           | Frontend plugin                                 | Entity page component that fetches and displays the OPA policy associated with an entity via a catalog annotation.                                                                                                        |
| `plugins/opa-common`                                    | `@parsifal-m/backstage-plugin-opa-common`                   | Shared library                                  | Shared TypeScript types used across multiple plugins (`PolicyInput`, `PolicyResult`, etc.).                                                                                                                               |

### Internal (demo only, not published)

| Directory                   | npm package                                    | Purpose                                                                 |
| --------------------------- | ---------------------------------------------- | ----------------------------------------------------------------------- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Parsifal-M/backstage-opa-plugins](https://github.com/Parsifal-M/backstage-opa-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
