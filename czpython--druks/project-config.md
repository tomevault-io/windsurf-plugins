---
trigger: always_on
description: Druks runs durable agent apps on DBOS and Postgres. It owns workflow execution,
---

# AGENTS.md

Druks runs durable agent apps on DBOS and Postgres. It owns workflow execution,
persisted state, events, gates, webhooks, sandbox access, and the shared
dashboard. Apps are standalone Python packages. They register through the
`druks.apps` entry point. `software_factory` is the bundled reference app for
coding work through GitHub pull requests.

## Read map

Start with `README.md`, then read only the material relevant to the task:

- Workflow lifecycle, state, replay, or recovery: `docs/concepts.md`
- App contracts or the public author surface: `docs/writing-an-app.md`
- App pages, blocks, values, fields, actions, and liveness: `docs/druks-ui.md`
- Configuration or environment variables: `docs/configuration.md`
- Local install and operations: `docs/full-local.md`
- Remote deployment: `docs/deployment.md`
- Failure diagnosis: `docs/troubleshooting.md`
- Backend contribution, migrations, or verification: `docs/development.md`
- Migration head: `alembic heads`, not a scan of `backend/migrations/versions/`
- Shared SPA work: `frontend/README.md`
- Documentation navigation and audience ownership: `docs/index.md`
- The checklist and craft gate every change is held to: `.druks/review/checklist.md`.

For app-surface changes, inspect the proof app at
`backend/tests/druks-field_notes/` and its tests as well as the author guide.

## Architectural boundaries

- Keep platform and app ownership explicit. Keep GitHub issue, branch, PR, and
  coding-agent policy in `software_factory`, not in Druks core.
- Describe durability precisely. Druks reuses completed durable checkpoints
  during replay. An interrupted operation can run again. Do not imply
  arbitrary-line resume or exactly-once external side effects.
- Derive `Run.state` from DBOS workflow status. Do not add a second writable state mirror.
- Import the public concern namespaces documented in `docs/writing-an-app.md`.
  Do not import Druks internals.
- Keep backend app discovery in runtime packaging. Keep shared-dashboard app UI
  registration in the frontend build. Keep standalone `dist/` delivery separate.
- Keep generic agent, harness, workspace, sandbox, event, gate, webhook, and
  settings plumbing in Druks. Keep domain-specific policy in the app.
- One Druks installation serves one organization. Execution defaults and agent
  overrides are shared. Accounts own credentials, personal preferences, and run
  attribution. Do not add personal execution defaults.
- Grow the author surface by parameter, not by namespace. If the SDK lacks a
  capability, widen the primitive that owns it. Add a keyword argument or a
  method to the class that holds the data. Do not add a namespace, facade,
  context object, or one-use helper module.
- Do not import an app from an author-surface module. Import only
  `druks.workflows`, `druks.workspaces`, `druks.agents`, `druks.events`,
  `druks.signals`, `druks.db`, `druks.schemas`, `druks.prompts`, `druks.durable`,
  `druks.apps`, `druks.ui`, and `druks.webhooks`. A reference to `druks.build` or another app in these modules
  inverts the platform. The builtin apps `core` and `chat` are platform code, so
  these modules can import them.
- Internal code takes `session: AsyncSession` as a required parameter. Only
  author code and the seams that bind the registry read `druks.db.db_session()`;
  ruff's banned-api names the seams in `pyproject.toml`.
- Derive liveness from run state. Do not mirror it in a column. Store an external
  outcome after its owner announces it. Do not infer that outcome from run
  lifecycle. A merge can occur after Druks stops the related run.
- Use `@step` to mark a replay checkpoint, not an expensive call. On replay the body
  re-executes from the top and completed steps return cached results, so code outside a
  step runs again. Moving the boundary changes correctness, not performance.
- Store transient live state and mutual exclusion in Redis, keyed by run id.
  Do not give either one a durable column, row lock, or in-process lock.
- Do not add a store before something reads it.
- Route a workflow cadence or pause through its schedule overrides. Do not use a
  settings column for this purpose.
- Use one canonical name and shape for each contract. Fail loudly on every other shape.
- Do not type-switch over a typed stream in app code. When a projection needs
  ordering or anchoring, grow the SDK primitive instead of an `isinstance` chain.
- Put only identity and facts in a read-side field. Keep UI wording in the app pages.
- Give a shared resource one global registry. When a second
  consumer needs a different answer, add a scoping axis. Do not add one in anticipation.

## Layout

- **Backend:** `backend/druks/` contains FastAPI, DBOS, SQLAlchemy 2.0,
  Pydantic v2, and bundled apps.
- **Migrations:** `backend/migrations/` contains platform Alembic migrations.
- **Tests:** `backend/tests/` contains the pytest suite backed by real Postgres.
- **Proof app:** `backend/tests/druks-field_notes/` contains the independently
  packaged proof app.
- **Frontend:** `frontend/` contains the React 19 and Vite shared SPA. The
  backend image includes its repository-root `dist/` output.
- **Deployment:** `deploy/` contains Compose files, Caddy configuration, and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [czpython/druks](https://github.com/czpython/druks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
