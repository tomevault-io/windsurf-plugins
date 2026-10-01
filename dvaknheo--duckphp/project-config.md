---
trigger: always_on
description: > Audience: humans and AI coding agents working in **this project** (a DuckPHP application).
---

# AGENTS.md — DuckPHP project conventions

> Audience: humans and AI coding agents working in **this project** (a DuckPHP application).
> Project-local conventions live here. The framework docs ship inside the package, both Chinese:
> `vendor/dvaknheo/duckphp/docs/zh/guide/` (how-to, 47 chapters) and
> `vendor/dvaknheo/duckphp/docs/zh/reference/` (per-class API and option defaults).
> **Never copy signatures or option tables into this file** — they drift. Keep it under ~200 lines.

## 1. Layout

```text
project/
├── public/index.php                  # web entry — do not edit
├── bin/cli.php                       # CLI entry — do not edit
├── config/DuckPhpSettings.config.php # DB / Redis settings (secrets live here)
├── AGENTS.md                         # this file
├── CLAUDE.md                         # pointer to AGENTS.md (for Claude Code)
├── runtime/                          # writable: logs, caches (not in version control)
├── src/
│   ├── System/
│   │   ├── App.php                   # the app class: every option is set here
│   │   ├── * ProjectException.php    # project exception base (disabled by default)
│   │   ├── * BusinessException.php   # thrown by Helper::BusinessThrowOn() (disabled by default)
│   │   └── * ControllerException.php # thrown by Helper::ControllerThrowOn() (disabled by default)
│   ├── Controller/
│   │   ├── Base.php                  # do not edit
│   │   ├── Helper.php                # do not edit (or use the framework's ControllerHelper)
│   │   ├── MainController.php        # welcome page and short routes
│   │   ├── Session.php               # every session read/write goes through this class
│   │   ├── * AppAction.php           # Action that the System layer calls
│   │   ├── * CommandAction.php       # CLI commands (disabled by default)
│   │   ├── * ExceptionAction.php     # exception reporter (disabled by default)
│   │   ├── * SomeAction.php          # sample Action — delete me
│   │   └── * testController.php      # sample controller, route /test/done — delete me
│   ├── Business/
│   │   ├── Base.php                  # do not edit
│   │   ├── Helper.php                # do not edit
│   │   ├── * DemoBusiness.php        # sample — delete me
│   │   └── * SomeService.php         # sample Service — delete me
│   └── Model/
│       ├── Base.php                  # do not edit; brings Db() / find() / add() / getList()
│       └── DemoModel.php             # sample — delete me
└── view/
    ├── main.php                      # sample view for the welcome page
    ├── test/done.php                 # sample view for the route /test/done
    └── _sys/
        ├── error_404.php
        └── error_500.php
```

A `*` marks a sample file or a feature **disabled by default**: delete what you do not use, or enable
one by uncommenting the matching option in `src/System/App.php`. `runtime/` must stay writable.

## 2. Naming

| Kind | Pattern | Example | Notes |
|---|---|---|---|
| Controller | `{Name}Controller` | `NoteController` | URL segment = method name |
| Controller method | `{action_prefix}{name}` | `index()` | prefix is empty by default, see §4 |
| Action (controller-layer reuse) | `{Name}Action` | `ExportAction` | needs an empty `__construct()` |
| CLI method | `command_{name}` | `command_sync()` | `php bin/cli.php sync` |
| Business | `{Name}Business` | `NoteBusiness` | entry point of business logic |
| Service (business-layer reuse) | `{Name}Service` | `MailService` | called by Business, not by Controller |
| Model | `{Name}Model` | `NoteModel` | class name picks the table: `NoteModel` → `note` |
| Session | `Session` | `Session` | the only place that reads/writes the session |
| Exception | `{Name}Exception` | `ProjectException` | project exceptions live in `System/` |

## 3. Layering rules

### 3.1 Who may call whom

| Caller ↓ / Callee → | Controller | Business | Model |
|---|---|---|---|
| **System** | ✅ **only via `*Action`** | ❌ — go through an Action | ❌ — go through an Action |
| **Controller** | ⚠️ same layer, via `*Action` | ✅ | ❌ |
| **Business** | ❌ | ⚠️ same layer, via `*Service` | ✅ |
| **Model** | ❌ | ❌ | ⚠️ cross-database model only |

Things the table cannot show:

- **`Db`** — only Model touches the database: `Model\Base` gives you `Db()`, `find()`, `add()` and
  `getList()`. A `Helper::Db()` call or raw SQL in Controller / Business / Service / View is out of
  bounds; System only does connection-level work while wiring (e.g. `Helper::DbCloseAll()`).
- **`Session`** — only the Controller layer, through `Controller\Session`: its trait's
  `get()/set()/unset()` are **protected**, so add a public method there and call
  `Session::_()->yourMethod()` from elsewhere. System touches it only while wiring the login
  systems; Business / Service / Model / View never do.
- **`Service`** — not a layer: business-layer reuse, called by `*Business`, never by a Controller.
- **View** — the controller hands data over with `Helper::Show($data, $view)`; a view only displays
  (globals such as `__h()`, `__url()`), it never queries the database or calls Business.

### 3.2 The System layer must go through an Action

CLI commands, the exception reporter, event listeners and route hooks all live in Controller-layer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dvaknheo/duckphp](https://github.com/dvaknheo/duckphp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
