---
trigger: always_on
description: Security workflow orchestration platform. Visual builder + Temporal for reliability.
---

# ShipSec Studio

Security workflow orchestration platform. Visual builder + Temporal for reliability.

## Stack

- `frontend/` — React + Vite
- `backend/` — NestJS API
- `worker/` — Temporal activities + components
- `packages/` — Shared code (component-sdk, backend-client)

## Development

```bash
just init              # First time setup
just dev               # Start the active instance (default: 0)
just dev stop          # Stop the active instance (does NOT stop shared infra)
just dev stop all      # Stop all instances + shared infra
just dev logs          # View logs for the active instance
just help              # All commands
```

**Active instance**:

```bash
just instance show     # Print active instance number
just instance use 5    # Set active instance for this workspace
```

**Instance env files**:

```bash
just instance-init 5           # Initialize .instances/instance-5/*.env
just instance-env init 5       # Create from app/.env or app/.env.example
just instance-env update 5     # Re-apply instance-scoped vars
just instance-env copy 5 6     # Copy env setup from instance 5 -> 6
just instance-env show 6       # Show file status and computed values
```

**URLs**:

- Frontend: `http://localhost:${5173 + instance*100}`
- Backend: `http://localhost:${3211 + instance*100}`
- Temporal UI (shared): http://localhost:8081

Full details: `docs/MULTI-INSTANCE-DEV.md`

### Multi-Instance Local Dev (Important)

Local development runs as **multiple app instances** (PM2) on top of **one shared Docker infra stack**.

- Shared infra (Docker Compose project `shipsec-infra`): Postgres/Temporal/Redpanda/Redis/MinIO/Loki on fixed ports.
- Per-instance apps: `shipsec-{frontend,backend,worker}-N`.
- Isolation is via per-instance DB + Temporal namespace/task queue + Kafka topic suffixing + instance-scoped Kafka consumer groups/client IDs (not per-instance infra containers).
- The workspace can have an **active instance** (stored in `.shipsec-instance`, gitignored).
- Instance env files are stored at `.instances/instance-N/{backend,worker,frontend}.env` and can be managed with `just instance-env ...`.

**Agent rule:** before running any dev commands, ensure you’re targeting the intended instance.

- Always check: `just instance show`
- If the task is ambiguous (logs, curl, E2E, “run locally”, etc.), ask the user which instance to use.
- If the user says “use instance N”, prefer either:
  - `just instance use N` then run `just dev` / `bun run test:e2e`, or
  - explicit env override (`SHIPSEC_INSTANCE=N just dev ...`) for one-off commands.

**Ports / URLs**

- Frontend: `5173 + N*100`
- Backend: `3211 + N*100`
- Temporal UI (shared): http://localhost:8081

**E2E tests**

- E2E targets the backend for `SHIPSEC_INSTANCE` (or the active instance).
- When asked to run E2E, confirm the instance and ensure that instance is running: `SHIPSEC_INSTANCE=N just dev start` (or `just instance use N` then `just dev start`).

**Keep docs in sync**

If you change instance/infra behavior (justfile/scripts/pm2 config), update `docs/MULTI-INSTANCE-DEV.md` and this section accordingly in the same PR.

### After Backend Route Changes

```bash
bun --cwd backend run generate:openapi
bun --cwd packages/backend-client run generate
```

### Testing

```bash
bun run test           # All tests
bun run typecheck      # Type check
bun run lint           # Lint
```

### Database

```bash
just db-reset                              # Reset database
bun --cwd backend run migration:push       # Push schema
bun --cwd backend run db:studio            # View data
```

## Rules

1. TypeScript, 2-space indent
2. Conventional commits with DCO: `git commit -s -m "feat: ..."`
3. Tests alongside code in `__tests__/` folders
4. **E2E Tests**: Mandatory for significant features. Place in `e2e-tests/` folder.
5. **GitHub CLI**: Use `gh` for all GitHub operations (issues, PRs, actions, releases). Never use browser automation for GitHub tasks.

### Frontend: Read Before Writing Code

Before writing ANY frontend code that fetches data or adds a page, you MUST read these files first:

1. `frontend/docs/state.md` — Decision guide: TanStack Query vs Zustand, hook patterns, anti-patterns
2. `frontend/docs/performance.md` — Stale time tiers, bundle splitting, prefetch patterns, query key architecture
3. `frontend/src/lib/queryKeys.ts` — Existing query key factories (add new keys here, never inline)
4. Browse `frontend/src/hooks/queries/` — Follow existing hook naming conventions (`use<Domain>Queries.ts`)

### Frontend Data Fetching (Mandatory)

6. **All API data must use TanStack Query hooks** in `frontend/src/hooks/queries/`. Never use `useState` + `useEffect` to fetch backend data — this is the single most important frontend rule.
7. **Query keys** go in `frontend/src/lib/queryKeys.ts` (org-scoped, factory functions).
8. **After mutations**, invalidate the relevant query cache via `queryClient.invalidateQueries()` — do not manually update local state.
9. **Derive data** from query results using `useMemo`, not by copying into separate `useState`.
10. **Zustand stores** are for client-only UI state (canvas, timeline, auth). Never store API data in Zustand.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ShipSecAI/studio](https://github.com/ShipSecAI/studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
