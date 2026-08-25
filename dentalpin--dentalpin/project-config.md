---
trigger: always_on
description: Working notes for AI agents on DentalPin.
---

# CLAUDE.md

Working notes for AI agents on DentalPin.

## Project

DentalPin — open-source dental clinic management software with a modular plugin architecture.

| Component | Tech |
|-----------|------|
| Backend | FastAPI (Python 3.11+), SQLAlchemy 2.0, Alembic |
| Frontend | Vue 3, Nuxt 3, Nuxt UI, TypeScript |
| Database | PostgreSQL (asyncpg) |
| Auth | JWT (access + refresh) |
| Container | Docker Compose |

License: BSL 1.1 (converts to Apache 2.0 after 4 years).

---

## Modular architecture (read first)

DentalPin is built as independent modules under `backend/app/modules/<name>/` with matching Nuxt layers. Treat the boundary as a contract.

**Hard rules:**
- Respect module isolation. Do **not** create cross-module dependencies that are not declared in the module's `manifest.depends`.
- Prefer the **event bus** for cross-module reactions. Direct service-to-service imports across modules are forbidden unless the target is in `depends`.
- Cross-module FKs are allowed **only** when the target is in `depends`. CI rejects migrations otherwise.
- Each module owns its Alembic branch (`branch_labels = ("<name>",)`). Never thread one module's revisions through another's chain — uninstall safety depends on it (issue #56).
- Permissions are namespaced: a module returns `resource.action` from `get_permissions()`; the registry prefixes with the module name.

**Before adding a feature, read `docs/technical/creating-modules.md`** — it is the source of truth for module structure, lifecycle, manifest, slots, events, tools/agents, and migrations.

**Engineering posture:**
- Think deeply before coding. Avoid over-engineering. Build only what the task needs.
- Refactor opportunistically when you spot duplication or drift, but keep refactors scoped — no drive-by rewrites.
- Do not introduce tech debt. If a shortcut is unavoidable, surface it explicitly in the PR description.

---

## When adding X, do Y (agent checklists)

| Trigger | Required actions |
|---------|------------------|
| New module | Create `backend/app/modules/<name>/CLAUDE.md` + `CHANGELOG.md`. Create `docs/technical/<name>/{overview,events,permissions}.md`. If the module has Nuxt pages, also `docs/user-manual/{en,es}/<name>/{index.md, screens/<slug>.md}` per page. Run `python backend/scripts/generate_catalogs.py`. Follow `docs/checklists/new-module.md`. |
| New screen (page under `<module>/frontend/pages/**`) | Create both `docs/user-manual/en/<module>/screens/<slug>.md` and `docs/user-manual/es/<module>/screens/<slug>.md` with the [frontmatter contract](./docs/technical/documentation-portal.md#2-frontmatter-contract-the-part-claude-relies-on). Screenshots into `docs/screenshots/<module>/`. |
| New endpoint | Document under the gating permission's row in `docs/technical/<module>/permissions.md`. Bump `last_verified_commit` on every screen MD whose `related_endpoints` covers it. |
| New event published or consumed | Add to `EventType` in `backend/app/core/events/types.py`. Add row to `docs/technical/<module>/events.md`. Re-run `generate_catalogs.py`. Document publisher payload in module CLAUDE.md. |
| New permission | Return from `get_permissions()` (no module prefix). List in `manifest.role_permissions`. Add row to `docs/technical/<module>/permissions.md`. Add to `frontend/app/config/permissions.ts` if user-facing. |
| New agent-exposed capability | Declare a `Tool` in `backend/app/modules/<name>/tools.py` and return it from the module's `get_tools()`. **Wrap an existing service method — never duplicate business logic.** Filter by `ctx.clinic_id`. Set `permissions=[...]` to the gating RBAC string the HTTP route uses, and `category` conservatively (`WRITE` for mutations, `DESTRUCTIVE` for deletes/irreversible side-effects). Set `exposes_free_text=True` if the result is free prose (it is then excluded from the cloud LLM path under redaction). **Return native values (UUID/Decimal/datetime/Pydantic) — `jsonify` at the registry chokepoint coerces them; don't hand-`str()`/`.isoformat()`/`float()`.** Name PII fields with redactor-known keys (`full_name`, `phone`, `email`, `dni`, `*_id`) so they tokenize. Document it under "Tools exposed" in the module CLAUDE.md. See [`docs/technical/copilot-agentic-architecture.md`](./docs/technical/copilot-agentic-architecture.md) §3. |
| Touched a screen's behaviour or visuals | Update the matching screen MD in **both** `docs/user-manual/{en,es}/<module>/screens/`. Refresh screenshots if visuals changed. Bump `last_verified_commit` in each locale. |
| Architectural decision | Copy `docs/adr/TEMPLATE.md` → `docs/adr/NNNN-title.md`. |
| New domain term (ES↔EN) | Append to `docs/glossary.md`. |
| New documentation file | Pick the folder by type from the **Documentation policy** table below. Never drop new files at `docs/` root. |
| Touched any module | Update its `backend/app/modules/<name>/CHANGELOG.md` under `## Unreleased`. |
| Cross-module FK or import | Target module MUST be in `manifest.depends`. CI rejects otherwise. |

Full docs-update recipe: [`docs/checklists/updating-docs.md`](./docs/checklists/updating-docs.md).
Architecture + rationale: [`docs/technical/documentation-portal.md`](./docs/technical/documentation-portal.md) ([ADR 0009](./docs/adr/0009-documentation-portal.md)).

Reference material:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dentalpin/dentalpin](https://github.com/dentalpin/dentalpin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
