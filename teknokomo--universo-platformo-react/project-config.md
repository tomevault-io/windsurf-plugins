---
trigger: always_on
description: ATTENTION!!! These are your basic rules of work, always take them into account !!! It is forbidden to perform any actions without taking into account these basic rules of work !!! Only taking into account these basic rules of work allows you to work efficiently and do what the user needs !!! Thanks to these basic rules, you get the right context for your work.
---

ATTENTION!!! These are your basic rules of work, always take them into account !!! It is forbidden to perform any actions without taking into account these basic rules of work !!! Only taking into account these basic rules of work allows you to work efficiently and do what the user needs !!! Thanks to these basic rules, you get the right context for your work.

# Custom Modes

1. If the user message starts with **VAN** command → `.gemini/rules/custom_modes/van_mode.md`
2. If the user message starts with **RESEARCH** or **RPLAN** command → `.gemini/rules/custom_modes/research_mode.md`
3. If the user message starts with **PLAN** command → `.gemini/rules/custom_modes/plan_mode.md`
4. If the user message starts with **CREATIVE** or **DESIGN** command → `.gemini/rules/custom_modes/creative_mode.md`
5. If the user message starts with **IMPLEMENT** or **IMP** command → `.gemini/rules/custom_modes/implement_mode.md`
6. If the user message starts with **QA** command → `.gemini/rules/custom_modes/qa_mode.md`
7. If the user message starts with **REFLECT** or **REF** command → `.gemini/rules/custom_modes/reflect_mode.md`
8. If the user message starts with **ARCHIVE** or **ARC** or **ARH** command → `.gemini/rules/custom_modes/archive_mode.md`
9. If the user message starts with **DOCS** or **DOC** command → `.gemini/rules/custom_modes/docs_mode.md`
10. If the user message starts with **GIT PULL** or **PULL** command → `.gemini/rules/custom_modes/git_pull_mode.md`
11. If the user message starts with **GIT PUSH** or **PUSH** command → `.gemini/rules/custom_modes/git_push_mode.md`
12. If the user message starts with **MB** command → `.gemini/rules/custom_modes/mb_mode.md`
13. If the user message starts with **DEVOPS** or **DEPLOY** command → `.gemini/rules/custom_modes/devops_mode.md`

All these Custom Modes use the rules that are in `.gemini/rules/`

# Runtime UI UX Quality Gate

When work touches MUI runtime screens, app-template dashboards, metahub template UI metadata, CRUD dialogs, DataGrid/table/card displays, relation builders, resource-source fields, or UI E2E flows, use the project-local skills `.agents/skills/mui-runtime-ux-patterns` and `.agents/skills/runtime-ux-qa`.

Non-negotiable runtime UI rules:

-   No raw user-facing IDs or hidden-knowledge workflows on normal user surfaces.
-   No raw JSON, `[object Object]`, or object cells in normal tables/cards.
-   Semantic long-text fields are multiline by default.
-   Validation messages are localized and user-facing.
-   Reuse existing MUI dashboard/app-template primitives before creating new UI.
-   Implemented UI needs browser UX evidence, including no page-level horizontal overflow.

# Repository Guidelines

## Project Structure & Module Organization
- Monorepo with feature apps under `packages/`.
  - Examples: `packages/publish-frontend` (React front end), `packages/publish-backend` (Node/Express back end), `packages/updl` (UPDL tools).
  - Each app contains a `base/` directory for the default implementation.
- Front-end apps include `i18n/` with default locales `en/` and `ru/`.
- Context docs and planning live in `memory-bank/` (`productContext`, `techContext`, `progress`, `tasks`).
- **Template system**: Two built-in metahub templates — `basic` (minimal widgets) and `basic-demo` (full demo with sample entities). Template data lives in `packages/metahubs-backend/base/src/domains/templates/data/`.
- **Create options**: `POST /metahubs` accepts optional `createOptions` (`createHub`, `createCatalog`, `createSet`, `createEnumeration` — all default true) to control which default entities are seeded.
- **Entity settings**: Five entity detail views (Hub, Catalog, Set, Enumeration, Publication) include a "Settings" tab that opens an edit dialog overlay via `EntityFormDialog`.

## Build, Test, and Development Commands
- `pnpm install`: Install workspace dependencies.
- `pnpm dev`: Start development servers (run from the target app directory when applicable). Important: due to repo size and resource usage, only the user should run this locally; agents must not run it automatically.
- `pnpm --filter <package> build`: Build a single package to validate it quickly (e.g., lint/type errors). Note: changes are fully applied across the workspace only after a full root rebuild.
- `pnpm build` (root): Full workspace rebuild; required to propagate changes (even for a single package) and ensure cross-dependency consistency.
- `pnpm start`: Run production server(s) for built apps.
- `pnpm lint`: Run ESLint across the workspace. For checking specific packages, use `pnpm --filter <package> lint` (e.g., `pnpm --filter publish-frontend lint`) to avoid long execution times. Run global lint only when necessary and with user approval.

## Coding Style & Naming Conventions
- Prefer TypeScript where present; otherwise modern ES modules.
- Indentation: 2 spaces; avoid trailing whitespace.
- React: `PascalCase` components, `camelCase` hooks/utils, `kebab-case` folders/files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [teknokomo/universo-platformo-react](https://github.com/teknokomo/universo-platformo-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
