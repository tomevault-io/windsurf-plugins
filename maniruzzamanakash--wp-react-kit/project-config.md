---
trigger: always_on
description: Guidance for any AI coding agent (Cursor, Claude Code, Codex, Copilot, etc.)
---

# AGENTS.md

Guidance for any AI coding agent (Cursor, Claude Code, Codex, Copilot, etc.)
working in this repository. `CLAUDE.md` defers to this file.

## Project

**WP React Kit / "Job Place"** — a WordPress plugin with a React admin UI
(`src/`) backed by a PHP REST API (`includes/`). It manages job postings
(CRUD, publish/draft, dashboard stats) inside `wp-admin`, and ships a
**public job board** built with Gutenberg blocks and the Interactivity API.

- Text domain: `jobplace`
- PHP namespace root: `Akash\JobPlace` (PSR-4 → `includes/`)
- REST namespace: `job-place/v1`
- Block namespace: `wrc/*` (e.g. `wrc/jobs-list`, `wrc/job-page`)
- Build output: `build/` (committed-ignored; produced by `@wordpress/scripts`)

## Commands

```bash
npm run build        # Production build (webpack) → build/
npm run start        # Watch/dev build
npm run test:unit    # Jest unit tests (jest-unit.config.js)
npm run makepot      # Regenerate languages/jobplace.pot
php -l <file>        # Lint a PHP file
```

After editing `src/`, run `npm run build` and confirm it compiles. After
editing PHP, run `php -l` on changed files.

## Directory map

```
job-place.php                 # Bootstrap: DI container, hooks, activation
includes/                     # PHP (PSR-4: Akash\JobPlace\*)
  Abstracts/                  # BaseModel, RESTController, DBMigrator
  Admin/                      # Admin menu registration
  Assets/Manager.php          # Enqueues scripts/styles (incl. DataViews CSS)
  Blocks/                     # PatternRegistry, BlockTemplatesService, EditorData
  Common/Keys.php             # Option-name constants
  Databases/
    Migrations/               # dbDelta schemas (Jobs, JobType, Company, …)
    Seeder/                   # Demo data
  Jobs/                       # Domain: Job model, JobStatus, Manager (queries)
  REST/                       # Controllers + Api.php (route registry)
  Routing/                    # Job detail permalinks + template router
  Setup/                      # Installer, Upgrader, PageSeeder
  Common/Settings.php         # Global plugin settings (option jobplace_settings)
  WordPress/Pages/            # PageService (default Jobs page)
  Traits/                     # InputSanitizer, Queryable
  blocks-interactivity.php    # Interactivity API script module registration
src/                          # React/TypeScript admin app + block sources
  index.tsx                   # Entry: mounts <App/> into #jobplace
  App.tsx                     # HashRouter + routes; runs useMenuFix
  routes/                     # Route table (path → page component)
  pages/                      # Route pages (HomePage, jobs/JobsPage, …)
  components/                 # UI (jobs/, dashboard/, Notices.tsx)
  blocks/                     # Gutenberg blocks (jobs board + job fields)
  scripts/                    # Interactivity modules (@jobplace/jobs, …)
  styles/                     # Shared front-end SCSS (jobs-board.scss)
  data/jobs/                  # wp-data Redux store (see below)
  interfaces/                 # Shared TypeScript types
  hooks/  utils/  style/      # Helpers, MenuFix, global SCSS
templates/
  patterns/                   # Block patterns (*.php → register_block_pattern)
  templates/                  # Plugin block templates (single-job.html)
```

> Note: several legacy folders under `src/components/` are empty leftovers from
> the starter kit (button, table, modal, inputs, …). Don't reintroduce them —
> use `@wordpress/components` / `@wordpress/dataviews` instead.

## Admin app (wp-admin)

- React via `@wordpress/element`; routing via `react-router-dom` **HashRouter**.
- Lists render with `@wordpress/dataviews` (`DataViews`); forms with `DataForm`.
- **State** lives in a single `@wordpress/data` store `wp-react/jobs`
  (`src/data/jobs/`):
  - `actions.ts` — action creators + generator actions (`setFilters`, `saveJob`,
    `deleteJobs`). Generator actions yield `controls` and re-throw on error.
  - `controls.ts` — `apiFetch` side effects (FETCH/CREATE/UPDATE/DELETE).
  - `resolvers.ts` — lazy data loaders for selectors (`getJobs`, `getJobStats`,
    `getJobDetail`, dropdowns). Resolved values are **cached**.
  - `selectors.ts`, `reducer.ts`, `default-state.ts`, `types.ts`, `endpoint.ts`.
- **Read data through the store**, not raw `apiFetch` in components.
- Jobs list company column uses `CompanyAvatar` (logo or initials placeholder;
  no broken images when Clearbit/external URLs fail).
- Notices: dispatch `@wordpress/notices` (`createSuccessNotice`/`createErrorNotice`
  with `{ type: 'snackbar' }`); `src/components/Notices.tsx` renders them.

## Job board blocks (front-end)

Built with `@wordpress/scripts` (webpack), **Interactivity API** for client-side
search/pagination without full page reloads, and PHP `view.php` render templates.

### Core blocks

| Block | Role |
|-------|------|
| `wrc/jobs-list` | Query root: toolbar, inner template, no-results, pagination |
| `wrc/jobs-template` | Repeats inner blocks per job (`<article class="jobplace-job-card">`) |
| `wrc/job-page` | Single job detail root (used in `single-job` template) |
| `wrc/job-title`, `wrc/job-company`, … | Dynamic job fields (context-driven) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ManiruzzamanAkash/wp-react-kit](https://github.com/ManiruzzamanAkash/wp-react-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
