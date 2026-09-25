---
trigger: always_on
description: FloCafe is an open-source, offline-first Electron desktop POS.
---

# FloCafe agent guide

FloCafe is an open-source, offline-first Electron desktop POS.

## Orientation & layout

- **Main process (`main/`):** Electron lifecycle and IPC (`main/index.ts`), Express API on `:3001` (`main/server.ts`), standalone KDS server on `:3002` (`main/kds.ts`), Server App on `:3003` (`main/server-app.ts`), SQLite database access via `better-sqlite3`, ESC/POS printing, and background services.
- **Frontend (`frontend/src/`):** Next.js 16 and React 19 application (statically exported via `output: 'export'` when `NEXT_BUILD_MODE=desktop`, or standard server runtime when unset), Zustand state, UI components, and translations.
- **Tests (`tests/`):** Backend unit, integration, and release test suites.
- **Documentation (`docs/`):** Design specs and audits (see [docs/README.md](docs/README.md)).
- **Workflows (`.github/`):** Issue/PR templates, CODEOWNERS, and CI/CD workflows.

## Progressive disclosure & source of truth

Before starting non-trivial work:
1. **Understand task scope:** Read the task and any linked issue/PR, then identify scope and acceptance criteria. For minor typos or isolated one-line edits, formal planning is not required.
2. **Consult documentation index:** Check [docs/README.md](docs/README.md) to locate relevant `CURRENT` or `ACTIVE DESIGN` documents. Documents marked `ACTIVE DESIGN` or `FORWARD-LOOKING` describe target architecture and may be ahead of current code; `HISTORICAL` docs provide context only.
3. **Check business decisions:** If the task touches authorization, access control, defaults, or other product-behavior rules, check [docs/business-decisions.md](docs/business-decisions.md) — it is a verifiable log of deliberate product decisions that a plausible-looking implementation can easily contradict. If a task seems to require deviating from an entry there, stop and confirm with the user rather than assuming the decision is stale.
4. **Inspect current code:** Active runtime code and automated tests define current behavior. If a task or design doc contradicts current code or references non-existent files, report the discrepancy rather than inventing unapproved architecture.
5. **Identify tests:** Locate existing coverage in `tests/`, `frontend/`, and any subsystem-local test directories relevant to the change.
6. **Plan and execute:** Keep changes focused on the approved task.

**Conflict precedence:** The approved task defines the intended change. Current code and tests define existing behavior. `AGENTS.md` and business decisions define boundaries the implementation must not violate.

## Core invariants

1. **Offline-first operation:** Core POS operation (orders, billing, KDS, printing) must function without internet connectivity. Optional network features (Google Drive, WhatsApp, cloud reporting) run only when explicitly configured and must fail gracefully when offline.
2. **Data safety:** Existing customer data must survive upgrades. Never reset, truncate, or drop user databases as a shortcut for migration design.
3. **Architecture boundaries:** UI language, tenant regional settings, and tax/compliance behavior are separate, decoupled domains.
4. **Business timestamps:** Persisted timestamps follow FloCafe's canonical storage conventions; configured store timezone applies to business-local presentation, day/shift boundaries, and reporting intervals.
5. **Backend authority:** Security-critical, payment, and tax calculations remain backend-authoritative.
6. **Orders are never ownership-gated:** FloCafe is an open system for order visibility — any staff role with order access can see and act on any order, regardless of who created it. Authorization is restricted by role (page/feature access) and by specific action (e.g. KDS stage transitions are chef/manager/owner-only, narrowed further by station/category assignment), never by comparing `order.user_id`/item creator against the current user. Accountability comes from audit attribution (every write is recorded against the authenticated actor), not from hiding orders between staff. Do not add or reintroduce a `role === 'server' && order.user_id !== user.userId`-style check anywhere in the backend; see `docs/business-decisions.md` and `docs/roles-and-permissions.md`.
7. **Reuse before adding:** Reuse existing helpers, utilities, and dependencies before introducing new packages.
8. **Scope discipline:** Implement only the approved task. Do not make opportunistic refactors across unrelated files.

## Sharp edges & operational rules

- **Desktop static export boundary:** When building for desktop (`NEXT_BUILD_MODE=desktop`), `frontend/` is exported as static HTML/CSS/JS (`output: 'export'`). In desktop mode, there is no runtime Next.js server-side execution, Next.js API routes, or server cookies; all dynamic backend logic belongs in Express (`:3001`) or Electron IPC. Standard Next.js server runtime (`next start`) applies only when `NEXT_BUILD_MODE` is unset (cloud mode).
- **Port contention on dev/test:** Daemons hold ports `:3001` (API), `:3002` (KDS), and `:3003` (Server App). If commands fail with `EADDRINUSE`, run `npm run clean` (`node kill-ports.js 3001 3002 3003`) to clear them before proceeding.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FreeOpenSourcePOS/FloCafe](https://github.com/FreeOpenSourcePOS/FloCafe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
