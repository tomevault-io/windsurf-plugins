---
trigger: always_on
description: Cloudflare-native headless CMS built with **Hono.js** + TypeScript on **Cloudflare Workers** / **D1**.
---

# SonicJS AI Development Guidelines

Cloudflare-native headless CMS built with **Hono.js** + TypeScript on **Cloudflare Workers** / **D1**.

## Core Technology Stack
- **Framework**: Hono.js
- **Runtime**: Cloudflare Workers
- **Database**: Cloudflare D1 (SQLite) with Drizzle ORM (legacy only — document layer is raw SQL)
- **Validation**: Zod
- **Testing**: Vitest (unit + real-SQLite) + Playwright (E2E)
- **Frontend**: HTMX + HTML tagged templates (admin UI)
- **Auth**: Better Auth (session + RBAC) — `c.get('user') === { userId, email, role }`
- **Deployment**: Wrangler

## Workspace boundary (Conductor)

- Work in `.conductor/hong-kong-v3/`. **Never read or write** `/Users/lane/Dev/refs/sonicjs/` (sibling main checkout).
- Target branch: `origin/v3`. PRs: `gh pr create --base v3`. Diff: `git diff origin/v3...`.

## Architecture direction: Document Model (authoritative)

The project is migrating from per-feature tables (`content`, `media`, `testimonials`, `email_log`, …) to a unified **document repository**. **All new features build on the document model. Do not add new feature tables.**

Full design + remediation runbook: `docs/ai/plans/document-model-poc-plan.md` (Appendix A = original design; §1–§7 = current implementation status, defect IDs `D1`–`D48`).

### The 5 document tables (migration `0002_documents.sql`)
1. **`document_types`** — registered schemas (code/plugin-owned).
2. **`documents`** — every content/media/plugin record + every historical version. Queryable scalar fields are exposed as **indexed JSON `VIRTUAL` generated columns** (`q_*`) on this table — `json_extract(data, '$.path')`.
3. **`document_references`** — typed strong/weak edges (FK to `documents`); powers "where used" and reference-aware delete.
4. **`document_facets`** — indexed rows for multi-valued scalar fields (e.g. `tags`) — the one case generated columns can't cover.
5. **`document_permissions`** — per-document ACL overrides on top of type-level base grants.

### Implementation rules (non-negotiable — each line is a bug already shipped)

| # | Rule |
|---|------|
| **R1** | All document writes use raw `env.DB.prepare(sql).bind(...)` inside `env.DB.batch([...])`. **Never** put Drizzle query-builder objects (`db.update()`, `db.insert()`) into a `batch`. |
| **R2** | `VIRTUAL` generated columns and partial / expression UNIQUE indexes live **only** in raw migrations (`0002`, `0003`, and future ALTERs via `MigrationService.ensureDocumentGeneratedColumns`). Never declare them in `db/schema.ts` (Drizzle can't express them). |
| **R3** | Every document read/write is tenant-scoped. Go through `DocumentRepository` (injects `this.tenantId`) or include `AND tenant_id = ?`. POC tenant = literal `'default'`. |
| **R4** | Document route handlers must not build raw document SQL. Use `DocumentRepository.list()` / `DocumentsService`. (Legacy `content`/`media` routes are the exception — and are being decommissioned.) |
| **R5** | Count placeholders/columns/binds by hand before committing any `INSERT`. Mock tests can't catch arithmetic bugs (R10). |
| **R6** | `version_number` is derived in SQL: `(SELECT COALESCE(MAX(version_number),0)+1 FROM documents WHERE root_id = ?)`. Never compute in JS — the partial unique index `idx_documents_unique_version` will reject concurrent collisions. |
| **R7** | Derived rows (`document_facets`, `document_references`) exist **only** for the current-draft and published rows of a root. Delete explicitly on supersede/unpublish — never rely on `ON DELETE CASCADE` (D1 FK enforcement is not guaranteed). |
| **R8** | Escape every user-controlled value rendered into HTML with `escapeHtml` from `utils/sanitize`. |
| **R9** | After editing any `packages/core/migrations/*.sql`: run `cd packages/core && npm run generate:migrations`, re-sync the `my-sonicjs-app/migrations/` copies (byte-identical), commit the regenerated `src/db/migrations-bundle.ts`. |
| **R10** | Unit tests on the pure-mock DB cannot verify SQL, constraints, batch atomicity, generated columns, or bind counts. Real coverage requires the `better-sqlite3` harness — `documents.sqlite.test.ts` + the `*.integration.test.ts` route harness. |
| **R11** | New E2E specs are numbered **68+** (highest existing is 67). |
| **R12** | POC runs **alongside** legacy paths. Do not drop legacy plugin tables in the POC — decommission only after read-flip + backfill + grep-gate (see plan §"Decommissioning"). |

### Key behaviors to preserve

- **Two axes**: `is_current_draft` and `is_published` are **separate** — a published doc stays live while an editor saves new drafts. `status` (`draft`/`published`/`archived`) is a derived UI label only.
- **Timestamps**: `documents.created_at`/`updated_at` are stored in **seconds** (legacy `content` used ms). Use `documentSecondsToMs()` from `services/documents.ts` at every response/render boundary that expects ms.
- **Pagination**: keyset on `(updated_at, id)` for all JSON APIs. `OFFSET` only allowed in admin HTML page-number tables, and only with an explicit comment.
- **ACL**: `isAllowed` precedence = **deny wins → explicit allow → base grants**. Authed callers' `principalSet` must include `{ type: 'role', id: <role> }` (base grants only match `public` + `role`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SonicJs-Org/sonicjs](https://github.com/SonicJs-Org/sonicjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
