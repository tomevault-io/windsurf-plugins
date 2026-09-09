---
trigger: always_on
description: Purpose: define the repo-wide implementation rules for AI agents working in this workspace.
---

# AGENTS.md - Workspace rules

Purpose: define the repo-wide implementation rules for AI agents working in this workspace.

Scope: these rules apply to the whole repository unless a deeper `AGENTS.md` file says otherwise.

Docs are the design source of truth. Follow the patterns in `docs/00-index.md`, `docs/01-modules.md`, and `docs/02-server.md` when deciding structure, naming, and layer boundaries.

never list sources or exclude files in a package.swift files a folder path should be sufficent.

never write sql code outside of a Database/AccessLayer folder. SQL code is only allowed within that folder.

never write functionality on a Scope object. Always use a repository or query for managing database models, always use the table representation objects and never use direct SQL code. 

never use typealias-es. never use tuples, except if user explicitly asks for this.

never create helpers or simplify existing patterns. prefer copy & paste code over introducing new things, never cross boundaries. 


## 1) Current project structure

This workspace is a multi-project Swift repository centered on a shared OpenAPI contract:

- `openapi`
  - Shared OpenAPI libraries used by clients and servers.
  - Contains the generated app and management API types.
- `application`
  - Modular backend runtime.
  - Executables: `Server`, `Migrator`, `Worker`, `WebApp`, `Static`.
  - Internal modules live under the top-level `modules/` directory.
- `application/Sources/WebApp`
  - Hummingbird-based web application target.
  - Feature code lives under the web app feature source root in feature-scoped folders.
  - Shared code lives under `Bootstrap`, `Core`, `Platform`, `Shared`, and `Views`.
- `application/Sources/Static`
  - Static-file server target.
- `application/Tests/WebAppTests`
  - Web application test target.
- `native-app`
  - Native client that consumes the shared OpenAPI contract.
- `docs`
  - Architecture and module notes.

Treat the code layout as the concrete structure and the docs as the pattern guide.

## 2) Design patterns to follow

### 2.1 Backend module pattern

Use the FATHOMS style described in the docs:

- `Domain`
  - Models, rules, `DomainError`, and repository/query ports.
- `Application`
  - `UseCase` implementations, DTOs, scopes, permissions/actions, and orchestration.
- `Infrastructure`
  - SQL-first table access, repository/query adapters, migration code, and technical helpers.
- `Server`
  - HTTP/OpenAPI assembly, routes, middleware, adapters, and request/response mapping.
- `Migrator`
  - Schema changes and seed data.
- `Worker`
  - Background jobs built from the same application ports and scopes.

Dependency direction must stay inward:

`Server` / `Migrator` / `Worker` -> `Infrastructure` -> `Application` -> `Domain`

### 2.2 Composition pattern

- Prefer module builders and a shared runtime dependency bag, as shown in `docs/02-server.md`.
- Keep wiring code in composition layers, not in use-case or domain code.
- Build use-cases from executors, scopes, and adapters.
- Keep per-feature wiring isolated and small.

### 2.3 API and mapping pattern

- Keep OpenAPI types in the shared OpenAPI package.
- Use generated request/response types at the HTTP boundary.
- Keep schema/DTO conversion in dedicated adapter or helper files.
- Keep handlers thin: parse input, call use-case, map output.

### 2.4 Web app feature pattern

- Keep each feature self-contained under the web app feature source root.
- Prefer the existing local structure in that feature:
  - `Domain`
  - `Application`
  - `Infrastructure`
  - `Presentation`
  - `Presentation/Views`
- Keep reusable cross-feature code in `Core`, `Platform`, `Shared`, or `Views` only when it is genuinely shared.

## 3) Core implementation rules

### 3.1 One object per file

- Always keep one primary object per file.
- "Object" means one `struct`, `class`, `enum`, `protocol`, or one tightly scoped `extension` family.
- If a file starts accumulating a second unrelated type, split it into a new file.
- Keep related helper types in separate files unless they are tightly coupled and genuinely small.
- Generated code can be exempt if the generator requires a different layout, but hand-written code should follow this rule.

### 3.2 File naming

- Use feature-oriented file names that describe the primary object.
- Prefer names that match the contained type or extension:
  - `AdminListUserRolePresenter.swift`
  - `UserInvitationForm.swift`
  - `ObjectToSchema+<Feature>.swift`
- Avoid vague names like `Helpers.swift` unless the file is truly shared and narrowly scoped.

### 3.3 Layer boundaries

- Do not leak transport-specific types into domain code.
- Do not put HTTP, SQL, or framework-specific behavior into `Domain`.
- Keep controllers, use-cases, repositories, queries, adapters, and mappings in their appropriate layer.
- Use repository/query ports in `Domain` or `Application`, with implementations in `Infrastructure`.

### 3.4 Mapping and adapter code

- Keep request/response mapping close to adapter layers.
- Avoid manual field-by-field mapping in handlers when a dedicated mapping helper is the existing pattern.
- Prefer small adapter files over large mixed-purpose files.

### 3.5 OpenAPI generation rule


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FeatherCMS/feather](https://github.com/FeatherCMS/feather) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
