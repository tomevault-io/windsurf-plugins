---
trigger: always_on
description: This file defines the engineering standards for contributors and coding agents working in this repository.
---

# AGENTS.md

This file defines the engineering standards for contributors and coding agents working in this repository.

Scope: entire repository.

## 1. Project Intent

`AmiyaEden` is a full-stack EVE Online operations platform with:

- Go backend under `server/`
- Vue 3 + TypeScript frontend under `static/`
- dynamic menu/routing
- role-based access control
- ESI / SSO integrations
- strongly typed frontend API contracts

Changes should preserve this structure. Consistency with the existing architecture is preferred over introducing new patterns.

## 2. Architecture Rules

### 2.1 Backend Layering

Backend flow must remain:

`router -> middleware -> handler -> service -> repository -> model`

Standards:

- `handler` is transport-only.
  - Parse request
  - read auth context
  - call service
  - return standardized response
- `service` owns business rules.
  - authorization decisions beyond simple route guards
  - orchestration across repositories
  - external API integration
  - data shaping for frontend use
- `repository` owns database access only.
  - no business policy
  - no HTTP calls
  - no Gin types
- `model` defines persistence and JSON contracts.
  - keep naming explicit
  - avoid ambiguous aliases

Do not put business logic in handlers or SQL shaping directly in handlers.

### 2.2 Frontend Layering

Frontend flow must remain:

`view -> api -> backend`

Supporting layers:

- shared logic in `static/src/hooks`
- app state in `static/src/store`
- route logic in `static/src/router`
- reusable UI in `static/src/components`
- type contracts in `static/src/types`

Standards:

- views should not call `fetch`/`axios` directly
- views should not duplicate backend contract types inline
- reusable table/list logic should prefer existing abstractions such as `useTable`
- routing, auth, and permission logic belongs in router/store/directives, not page-local hacks

## 3. API Contract Standards

The frontend and backend are tightly coupled. Keep contracts synchronized.

When changing an endpoint:

1. update backend response/request shape
2. update frontend API wrapper in `static/src/api`
3. update shared TS types in `static/src/types/api/api.d.ts`
4. update UI usage
5. update `DEVELOPER_API.md` if the public contract changed

Rules:

- prefer additive changes over breaking changes
- preserve field names unless there is a clear bug
- use explicit JSON field names
- if the backend returns `issued_at`, the frontend must use `issued_at`, not a guessed alias like `created_at`

## 4. Localization Standard

All user-facing text must be localized.

Required:

- no hard-coded Chinese or English strings in views, dialogs, tables, empty states, buttons, or toast messages
- add entries to both:
  - `static/src/locales/langs/zh.json`
  - `static/src/locales/langs/en.json`
- prefer existing namespaces before creating new ones

Allowed exceptions:

- developer comments
- internal debug logs
- seed/demo content only if clearly isolated and non-user-facing

Preferred pattern:

- template: `$t('namespace.key')`
- script: `const { t } = useI18n()` then `t('namespace.key')`

## 5. Backend Standards

### 5.1 Responses

Use the existing unified response helpers. Do not invent per-handler response envelopes.

### 5.2 Authorization

- coarse access control belongs in router/middleware
- fine-grained ownership/role checks belong in service
- do not rely on frontend-only authorization

### 5.3 Persistence

- repositories should query only what they need
- if the frontend needs enriched rows, prefer a dedicated DTO/view model instead of polluting a base persistence model
- keep query joins explicit and readable

### 5.4 External Integrations

- ESI/SSO calls belong in service or `pkg/eve`, not in handlers or repositories
- isolate retry/timeout behavior
- log failures with actionable context, not generic messages

## 6. Frontend Standards

### 6.1 Page Composition

- keep pages thin
- extract repeated UI into components
- extract repeated data behavior into hooks
- prefer computed/render helpers over duplicated inline formatting logic

### 6.2 State

- page-local state stays in the page
- shared cross-page state goes to Pinia
- do not put server-derived state into global store unless multiple routes need it

### 6.3 Tables and Forms

- use existing shared patterns (`ArtTable`, `ArtTableHeader`, `useTable`, shared dialogs) when possible
- keep column labels localized
- keep search placeholders localized
- keep validation messages localized

### 6.4 Routing and Menus

- preserve current dynamic route architecture
- do not hardcode route visibility assumptions into pages
- menu, permission, and route definitions must stay aligned

## 7. Type Safety Standard

- do not use `any` unless there is no practical alternative
- prefer existing `Api.*` types
- if a response is a special case, create a named interface or dedicated type
- keep backend and frontend field naming aligned exactly

## 8. Change Management Rules

Before editing:

- inspect the surrounding module first
- follow existing patterns in that slice of the codebase
- do not refactor unrelated areas opportunistically

When editing:

- keep changes scoped
- preserve backward compatibility where feasible
- avoid hidden coupling

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zifox666/AmiyaEden](https://github.com/zifox666/AmiyaEden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
