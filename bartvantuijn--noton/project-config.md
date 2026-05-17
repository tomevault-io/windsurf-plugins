---
trigger: always_on
description: Guidance for AI assistants working in this repository. `AGENTS.md` is the source of truth; tool-specific files such as `CLAUDE.md` should only point back here.
---

# AGENTS.md

Guidance for AI assistants working in this repository. `AGENTS.md` is the source of truth; tool-specific files such as `CLAUDE.md` should only point back here.

## Project overview

**Noton** is a free, self-hosted documentation platform built on **Laravel 12** and **Filament v4**, with private AI features powered by **Ollama** or **OpenClaw**. It is licensed under FSL-1.1 and distributed as a Docker image (`ghcr.io/bartvantuijn/noton`).

The product surface is basically a Filament admin panel mounted at the site root — there is no separate public frontend. Posts (Markdown documents) live inside nested Categories. Guests see public content; admins manage everything and can chat with an AI that is grounded in the documentation they have indexed.

## Tech stack

- **PHP** 8.2+ (CI and Docker build use 8.4)
- **Laravel** ^12.0
- **Filament** ^4.0 (panel builder + Livewire under the hood)
- **Livewire** (bundled with Filament)
- **Spatie Media Library** plugin (logos/favicons on `Setting`)
- **Spatie Tags** plugin (`Post` tags)
- **Phiki** for Markdown syntax highlighting
- **Flowframe Laravel Trend** for dashboard stats
- **Node** 24 + **Vite** 7 + **Tailwind CSS** 4 (`@tailwindcss/vite`)
- **PHPUnit** 11 for tests, **Mockery** for mocks
- **Pint** (Laravel preset + custom rules) and **Rector** for code style
- **SQLite** in-memory for tests; **PostgreSQL** or **MySQL** in production (configured via `DB_CONNECTION`)

## Repository layout

```
app/
├── Enums/                  # PHP 8.1 backed enums (e.g. Visibility)
├── Filament/
│   ├── Pages/              # Dashboard, Settings, Auth (Login, Register)
│   ├── Resources/          # CRUD resources per model (Categories, Posts, Users)
│   │   └── <Model>/{Pages,Schemas,Tables}
│   └── Widgets/            # Dashboard widgets (StatsOverview, MostViewedPosts…)
├── Helpers/                # Thin static helpers (App::hasUsers, etc.)
├── Http/Middleware/        # RedirectToLogin, RedirectToRegistration
├── Livewire/               # ChatModal (AI assistant)
├── Models/
│   └── Scopes/             # Global scopes (VisibleScope)
├── Observers/              # PostObserver
├── Policies/               # CategoryPolicy, PostPolicy, UserPolicy, SettingPolicy
├── Providers/              # AppServiceProvider, Filament/AdminPanelProvider
└── Services/               # OllamaService, OpenClawService
bootstrap/                  # app.php, providers.php
config/                     # Standard Laravel config (+ services.php for AI)
database/
├── factories/
├── migrations/
└── seeders/DatabaseSeeder.php
docker/                     # entrypoint.sh, supervisor/caddy configs
docs/setup.md               # User-facing setup guide
lang/                       # en/ + nl.json translations
resources/
├── css/app.css             # Tailwind entry
├── js/{app,bootstrap,main,jquery}.js
└── views/                  # Blade: filament/, livewire/, components/
routes/                     # web.php (empty — Filament handles routing), console.php
tests/
├── Feature/                # AuthRedirectTest, ChatModalTest, NestedCategoriesTest, VisibilityTest
├── Unit/                   # VisibilityTest
└── TestCase.php
```

## Architecture highlights

- **Filament is the whole app.** `AdminPanelProvider` mounts the panel at `path('')` (site root) and custom-builds the sidebar in `getNavigation()`: Dashboard + resource items, followed by dynamic navigation groups built from root `Category` records with their nested children and posts. There are no routes in `routes/web.php` — Filament discovers resources/pages/widgets under `app/Filament`.
- **Initial-user flow.** `RedirectToRegistration` forces the first visit to the registration page when `users` is empty, then hides registration once a user exists. The first registered user is expected to become the admin (`role = 'admin'`).
- **Guest vs. admin visibility.** `VisibleScope` is applied as a global scope to `Post` and `Category`. When there is no authenticated user, it hides private posts and all posts under any category that is private (directly or via ancestors). `RedirectToLogin` additionally guards edit/create/settings routes and redirects guests away from effectively-private view pages.
- **Nested categories.** `Category` has self-referencing `parent` / `children`. A `saving` hook (`validateParent`) prevents self-parenting and cycles. `getAncestors()` walks up without global scopes so the private-ancestor check works even when the scope would hide a parent.
- **Post content is Markdown.** Rendered with `Str::markdown()` and the Phiki CommonMark extension (`github-light-default` / `github-dark-default`) for code highlighting. `Post::summary()` strips tags and supports query highlighting for search results.
- **Tags** are Spatie tags on `Post`. `PostObserver::saved()` deletes any `Tag` with zero posts after a save (keeps the tag list tidy).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bartvantuijn/noton](https://github.com/bartvantuijn/noton) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
