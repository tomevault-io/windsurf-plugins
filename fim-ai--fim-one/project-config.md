---
trigger: always_on
description: FIM One is an AI-powered Connector Hub. Python async framework, provider-agnostic, protocol-first.
---

# FIM One — Claude Code Instructions

## Project Overview

FIM One is an AI-powered Connector Hub. Python async framework, provider-agnostic, protocol-first.

- **Package manager**: `uv` (not pip)
- **Frontend**: Next.js + pnpm (in `frontend/`)
- **Tests**: `uv run pytest`
- **Launcher**: `./start.sh [portal|api]`

## Architecture

```
src/fim_one/
├── core/
│   ├── agent/       # ReActAgent (JSON mode + native function calling)
│   ├── model/       # BaseLLM, OpenAICompatibleLLM, ModelRegistry, retry, rate limiting, usage tracking
│   ├── planner/     # DAGPlanner → DAGExecutor → PlanAnalyzer
│   ├── memory/      # WindowMemory, SummaryMemory
│   └── tools/       # Tool base classes, ConnectorToolAdapter
├── web/             # FastAPI backend API (agents, connectors, KB, chat)
frontend/            # Next.js portal (shadcn/ui)
```

## Git Rules (MANDATORY)

- **Commit scope**: (1) if the current session has context, commit all files involved in the session's tasks; (2) if this is a fresh session with no prior discussion, commit everything (`git add -A`); (3) if the user provides an inline instruction specifying which files to commit, follow that scope exactly. All three cases must exclude sensitive files (.env, credentials, etc.)
- **Atomic commits**: always split unrelated changes into separate commits, even if user says "commit all"
- **NEVER `git stash --include-untracked`** with important untracked files — `git add` them first; use `git stash pop` not `apply` + `drop`
- **Worktrees**: clean working tree before starting; agents MUST commit on their branch; merge via `git merge`/`git cherry-pick`, not file copying
- **Worktree + DB migration**: worktree agents MUST NOT run `alembic upgrade head` or any migration command — only write migration files + ORM model changes + business code. Migrations are applied after merging back to main (via `start.sh` auto-upgrade or manual `alembic upgrade head`). Reason: worktree shares the same SQLite dev DB; running migrations there will break the running main process and desync `alembic_version`.
- **Worktree merge-back**: after a worktree agent completes, the orchestrator (main conversation) MUST merge the feature branch back to the current branch via `git merge <branch>` — do NOT leave orphan branches or ask the user to merge manually

## Frontend Build Safety

- **NEVER `rm -rf frontend/.next`** while dev server is running — kills Turbopack HMR
- Production builds → `.next-build` (separate dir). To build: `cd frontend && pnpm build`

## Frontend UI Conventions

- **No native dialogs** (`confirm`/`alert`/`prompt`) — use `AlertDialog` / `Dialog` / Toast (sonner)
- **Navigation → `<Link>`**, not `<button onClick={router.push()}>`. For shadcn `<Button>`, use `asChild` + `<Link>` inside
- **Focus rings**: use `focus-visible:outline-*` pattern, **never** `focus-visible:ring-*` — `ring` is `box-shadow`, gets clipped by `overflow:hidden`. See `ui/input.tsx` for reference.
- **No `pl-*`/`pr-*` on form wrappers** containing `<Input>`/`<Textarea>` — clips `shadow-xs`. Use `px-*` or `gap-*` instead.
- **No native `<select>`** — use shadcn `<Select>` with `<SelectTrigger className="w-full">`. For empty defaults, use `__default__` sentinel (Radix treats `""` as unset).
- **Tab/filter state → URL query param**: any page with tabs or filter switching MUST sync to `?tab=xxx` via `useSearchParams` + `router.replace`. Default tab uses no param (clean URL). Wrap component in `<Suspense>`. See `admin/page.tsx`, `settings/page.tsx`, `artifacts/page.tsx` for reference.

## Admin List Page Actions Convention (MANDATORY)

All admin pages that display a data table **MUST** follow the `admin-users.tsx` pattern for row actions:

- **Actions MUST live in a "..." `DropdownMenu`** — one `MoreHorizontal` ghost button (`h-7 w-7 p-0`) in the last column, right-aligned (`text-right`)
- **NEVER expose inline icon buttons** (trash, edit, eye, power, toggle) directly in table rows
- **NEVER make rows clickable** (`cursor-pointer` + row `onClick`) — use a DropdownMenuItem instead (e.g. "View Details")
- **Column header**: last `<th>` must be `{tc("actions")}` with `text-right font-medium text-muted-foreground`
- **Every `DropdownMenuItem` MUST have a lucide icon** (`<Icon className="mr-2 h-4 w-4" />`) — no text-only menu items
- **Item order**: safe actions first (View, Edit), then state-toggle (Enable/Disable), then destructive last (Delete) with `variant="destructive"` and a `DropdownMenuSeparator` before it
- **Dialogs/Sheets stay as siblings** of the table — only the trigger moves into the dropdown, never the modal itself
- **Reference**: `admin-users.tsx` (full pattern), `admin-api-keys.tsx` (minimal pattern)

## Error Feedback Convention (MANDATORY)

**Two-tier strategy — inline for field errors, toast for system errors:**

- **Field-level validation errors** (empty required field, invalid format, value conflict like "username taken") → **inline `<p className="text-sm text-destructive">` below the field**. Use `fieldErrors` state + `clearFieldError` on input change. Add `aria-invalid` for accessibility.
- **System-level / API errors** (500, network timeout, auth failure, external service down) → **`toast.error(errMsg(err))`**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fim-ai/fim-one](https://github.com/fim-ai/fim-one) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
