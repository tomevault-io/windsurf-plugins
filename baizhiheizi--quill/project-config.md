---
trigger: always_on
description: > Context for AI coding agents working in this repository.
---

# AGENTS.md

> Context for AI coding agents working in this repository.

## Project Overview

Quill is a Web3 paid-publishing platform ([quill.im](https://quill.im/)) where authors publish priced articles and readers pay to access them. Its distinguishing feature is **early reader rewards**: a share of each article's new revenue (default 40%) is distributed proportionally to readers who paid earlier. The stack is a Rails monolith with Hotwire (Turbo + Stimulus), ERB partials, PostgreSQL, Solid Cable, Solid Cache, and background jobs via Solid Queue (separate queue database). Integrations include Mixin Network (OAuth) and MixPay (cross-asset payment rail).

## Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Language | Ruby | 4.0.5 (see `.ruby-version`, `mise.toml`) |
| Framework | Rails | 8.1.x |
| Database | PostgreSQL | — |
| Real-time | Solid Cable (separate `*_cable` DB) | — |
| Cache / jobs | Solid Cache, Solid Queue | — |
| Frontend | Turbo, Stimulus, Tailwind, esbuild | Node 20+, Bun 1.x |
| Testing | Minitest, Capybara | minitest ~> 6.0 (locked to 6.0.6) |
| Lint | RuboCop (rails-omakase), Prettier | — |
| Deploy | Kamal, Docker | manual `workflow_dispatch` |

## Architecture

Classic Rails MVC with namespaced controllers, route draws, service objects, and ActiveJob workers. Public site, author **dashboard**, **admin**, JSON **API**, and **grover** sub-apps share models but use separate controllers.

### Directory Structure

```
quill/
├── app/
│   ├── controllers/     # Web, dashboard, admin, api, grover
│   ├── models/          # ActiveRecord + concerns (AASM, etc.)
│   ├── views/           # ERB templates and partials
│   ├── helpers/         # View helpers (UiHelper for modal/dropdown wrappers)
│   ├── javascript/      # Stimulus controllers, esbuild entry
│   ├── jobs/            # Active Job work (orders, transfers)
│   ├── services/        # Query/command objects (e.g. ArticleSearchService)
│   ├── notifiers/       # Noticed 3 notifier classes + delivery_methods/
│   └── libs/            # Non-Rails Ruby helpers
├── config/
│   ├── routes/          # Route draws: admin, dashboard, api, grover
│   ├── settings/        # Config gem YAML (copy to settings.local.yml)
│   └── credentials/     # Encrypted secrets (Mixin bot, encryption keys)
├── db/migrate/          # Schema migrations
├── test/                # Minitest (models, controllers, jobs, notifiers)
└── .github/workflows/   # check.yml (CI), deploy.yml (Kamal)
```

## Development

### Setup

```bash
bundle install
bun install
EDITOR=vim bin/rails credentials:edit --development   # Mixin bot + AR encryption keys
cp config/settings.yml config/settings.local.yml       # edit host for local URL
bin/rails db:prepare
```

Requires PostgreSQL running locally (or via Docker). See `CONTRIBUTING.md` for credential field examples (note: README versions are authoritative over CONTRIBUTING's Ruby 3.2 note).

### Run

```bash
bin/dev   # Procfile.dev: Rails, Solid Queue (bin/jobs), CSS/JS watch, mixin_blaze
```

App: `http://localhost:3000`. Admin: `http://localhost:3000/admin` (create `Administrator` in console).

### Test

```bash
bin/rails db:setup   # or db:prepare
bin/rails test
bin/rails zeitwerk:check
```

CI also runs `bin/rubocop` and `bun run lint-check`.

### Benchmarks

```bash
bin/benchmark                    # all hot-path scenarios (test fixtures)
bin/benchmark article_search     # filter by scenario name
```

See `test/benchmarks/README.md` for env vars and limitations. Stdlib-only; not run in CI.

### Frontend efficiency

```bash
bin/measure-frontend-efficiency            # bundle sizes, lazy-loading & motion coverage, listener-leak sweep
bin/measure-frontend-efficiency --json     # machine-readable (for before/after diffs / future CI baselines)
bin/measure-frontend-efficiency --minify   # also measure minified JS (builds once; restores dev build)
```

Stdlib-only; no Rails boot. Gracefully reports `not built` / `null` when `app/assets/builds/` or `node_modules/` are absent. Not run in CI.

### Lint

```bash
bin/rubocop
bun run lint-check          # Prettier check on app/javascript
bun run lint                # Prettier write
```

### Build assets (without bin/dev)

```bash
bun run build
bun run build:css
```

## Code Conventions

- **Frozen string**: `# frozen_string_literal: true` at top of Ruby files
- **Naming**: snake_case files/methods; PascalCase classes; `API::` namespace for API controllers
- **Models**: schema annotations via `annotaterb`; AASM `state` columns; counter caches
- **Services**: class with `.call` factory (see `ArticleSearchService`)
- **Views**: reusable UI in `app/views/**/_*.html.erb` partials; block/slot patterns via `UiHelper` (`render_modal`, `render_dropdown`, etc.)
- **Controllers**: concerns in `app/controllers/concerns/` (`Localizable`, `RenderingHelper`, `API::RenderingHelper`)
- **Routes**: partial routes in `config/routes/*.rb`, loaded via `draw :name` in `config/routes.rb`
- **JS**: Stimulus controllers in `app/javascript/controllers/`; entry `app/javascript/application.js`
- **Comments**: sparse; schema comments auto-generated on models

## Testing Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [baizhiheizi/quill](https://github.com/baizhiheizi/quill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
