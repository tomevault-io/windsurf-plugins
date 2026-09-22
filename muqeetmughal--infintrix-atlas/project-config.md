---
trigger: always_on
description: - **Backend**: Frappe/ERPNext app (`infintrix_atlas/`), modifies standard `Project` & `Task` via custom fields + custom doc types (Cycle, Project Phase, Requirement, etc.)
---

# AGENTS.md — Infintrix Atlas

## Architecture

- **Backend**: Frappe/ERPNext app (`infintrix_atlas/`), modifies standard `Project` & `Task` via custom fields + custom doc types (Cycle, Project Phase, Requirement, etc.)
- **Frontend (legacy)**: React SPA (`atlas/`), served via `www/atlas.html` + `www/atlas.py` with boot context. Routes: Dashboard, Tasks, Projects, Team, Profile, AI Architect, Customer Portal. **Being deprecated** — new internal features must be built as native Frappe Desk pages (`page/`) instead.
- **Frontend (current)**: Native Frappe Desk pages and forms. Example: `project_backlog` Desk page replaces the React `/atlas` backlog view.
- **Frontend port**: `8080` (Vite dev server proxies to Frappe) — only for legacy React dev
- **Customer portal access**: based on `Customer.portal_users` membership, not a `Client` role
- **Core modeling**:
  - Cycle = execution planning (primary unit)
  - Project Phase = deprecated / being removed
  - Task status = execution state
  - Backlog is derived, not stored

## Developer commands

```bash
# Frontend dev (from atlas/ — separate terminal from bench)
yarn dev                  # vite on port 8080

# Frontend build + deploy
yarn build                # builds to infintrix_atlas/public/atlas/, then copies index.html to www/atlas.html

# Backend
bench migrate             # loads fixtures after python changes
bench --site sitename clear-cache   # required after JS/CSS changes in page/ or public/
bench watch               # auto-compile assets

# Lint
ruff check infintrix_atlas/  # Python (ruff config in pyproject.toml)
cd atlas && yarn lint        # JS (eslint)

# Pre-commit (CI)
pre-commit run --all-files
```

## Frappe v16 gotchas

- **Query builder rejects raw SQL strings in `.select()`**: `frappe.get_all(fields=["'Task' as type"])` fails. Use `frappe.db.sql(...)` or dict syntax instead.
- **`override_doctype_class` allows only one app per doctype**: HRMS already overrides `Project`. `AtlasProject` extends `EmployeeProject` but Frappe won't chain them automatically. Use `doc_events` for Project hooks instead.
- **`hooks.py` has duplicate `doc_events`** — only the last one takes effect. Keep merged.
- **Class name mismatch**: `overrides/project.py` has `AtlasProject` but hooks.py may reference `Project`. Fix the string.
- **After permission / override changes**: prefer `bench --site sitename clear-cache`, and restart bench workers/web if behavior still appears stale

## Permission model

- **Administrator**: full access (no filter)
- **System Manager**: full access
- **Projects Manager**: owned projects OR projects where user is in `Project User` child table
- **Projects User**: only projects where user is in `Project User` child table
- **Customer portal user**: project-linked access through matching `Project.customer` to `Customer.portal_users`
- Backend: `permissions.py` provides `permission_query_conditions` for project-linked lifecycle doctypes as well as Project/Task/Fathom
- `TaskOverride.has_permission` in `overrides/task.py`: admin, task owner, ToDo assignee, project owner, project member, or eligible customer portal user can view task detail

## React frontend conventions (Deprecated)

> **Do not build new features in the React SPA.** The `atlas/` React app is being deprecated. New UI work must be implemented as native Frappe Desk pages (`page/`) or standard DocType views.

- **Framework**: React 19 + React Router v7 + Antd v6 + `frappe-react-sdk` + Tailwind v4
- **React hooks must be unconditional**: No hooks after early return (e.g. `if (isLoading) return <Spin />`) — causes error #310
- **Calling backend**: Use `useFrappeGetCall`, `useFrappePostCall`, `useFrappeGetDoc` etc. from `frappe-react-sdk`
- **CSRF token**: Set as `window.csrf_token` from template. Dev mode fetches must include `X-Frappe-CSRF-Token: window.csrf_token` header
- **Auth guard**: `RequireRole` / `RoleGate` components in `components/auth/RequireRole.jsx`; `useHasRole` hook

## Backend API key endpoints (`infintrix_atlas.api.v1`)

| Endpoint | Purpose |
|---|---|
| `list_projects` | Filtered project list (respects Project User permission) |
| `list_tasks` | Filtered task list with group_by, permission-filtered |
| `backlog_with_phases` | Backlog grouped by phase + cycles per phase |
| `switch_assignee_of_task` | Reassigns task (closes old ToDo, creates new, auto-adds to Project User) |
| `set_project_mode` | Scrum/Kanban toggle (requires Project Manager role) |
| `get_user_roles` | Returns current user's roles |
| `get_project_user_stats` | Dashboard stats for user (uses `frappe.db.sql` — not `frappe.get_all`) |
| `get_customer_portal_data` | Dynamic customer-facing project view |
| `list_project_requirements` | Requirements for a project |
| `submit_portal_requirement` | Customer submits new requirement |
| `list_project_change_requests` | Change requests for a project |
| `submit_change_request` | Submit change request |
| `approve_change_request` | Approve change request and generate new requirement |
| `list_scope_snapshots` | Scope baselines for a project |
| `create_scope_snapshot` | Create new scope baseline |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [muqeetmughal/Infintrix-Atlas](https://github.com/muqeetmughal/Infintrix-Atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
