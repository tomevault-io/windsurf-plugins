---
trigger: always_on
description: Guidance for AI coding agents working on Total.js CMS.
---

# AGENTS.md

Guidance for AI coding agents working on Total.js CMS.

## Project

Total.js CMS is a content management system built on Total.js v5. It provides an admin interface, plugin-based content management, widget/layout/page rendering, and public site rendering.

Package name: `cms`
Main entry: `index.js`
Runtime dependency: `total5`
Default port: `8000`
License: MIT

## Repository Layout

- `index.js` starts Total.js v5 and calls `F.run(options)`.
- `config` contains application configuration such as `$api`, `salt`, and CDN settings.
- `definitions/` contains bootstrap, authentication, database initialization, and shared functions.
- `controllers/default.js` handles admin entry, public rendering, sitemap, page/layout compilation, redirects, and visitor script injection.
- `controllers/api.js` handles internal admin backup, restore, and cache clearing.
- `modules/` contains integrations such as CDN, OpenPlatform, and visitors.
- `views/admin.html` is the admin shell.
- `public/` contains admin/frontend assets, CMS editor UI, forms, pages, CSS, and JavaScript.
- `plugins/` contains built-in CMS plugins such as admin, pages, layouts, widgets, files, nav, variables, settings, dashboard, redirects, and posts.
- `--bundles--/` contains generated/bundled app and plugin artifacts used by Docker or bundled deployment.

## Commands

Install dependencies:

```bash
npm install
```

Run locally:

```bash
npm run start
```

Equivalent direct command:

```bash
node index.js 8000
```

Run in release mode:

```bash
node index.js 8000 --release
```

Run in service mode:

```bash
node index.js 8000 --service
```

Check JavaScript syntax for changed files:

```bash
node --check path/to/file.js
```

There is no test script in `package.json`; do not claim that `npm test` validates this project.

## Bundles and Docker

- `bundle.sh` is a Node.js bundle compiler script that writes `--bundles--/app.bundle` and optional plugin bundles.
- Built-in plugins such as dashboard, files, layouts, nav, pages, redirects, settings, variables, and widgets are included in the main app bundle.
- The Dockerfile copies `index.js`, `config`, `package.json`, `--bundles--/app.bundle`, and `--bundles--/admin.bundle`.
- Treat files in `--bundles--/` as generated deployment artifacts. Update them only when the task is specifically about bundled or Docker output.
- Do not manually edit bundle files when the source file can be changed instead.

## Coding Style

- Follow the existing Total.js v5 style in nearby files.
- Use framework globals already present in the codebase, such as `ROUTE`, `NEWACTION`, `AUTH`, `ON`, `CONF`, `MAIN`, `FUNC`, `FILESTORAGE`, `TRANSFORM`, `COMPONENTATOR`, and `OpenPlatform`.
- Prefer existing callback style and object shapes over introducing new abstractions.
- Keep route declarations compact and aligned with existing controller patterns.
- Avoid broad formatting-only changes.
- Do not add third-party dependencies unless the task explicitly requires them.

## Total.js v5 Rules

- Verify framework behavior in the local Total.js v5 source or existing CMS code before changing APIs or examples.
- Do not assume Total.js v4 APIs exist here.
- Use `NEWACTION()` patterns from plugin `schemas/` files.
- Use `ROUTE()` patterns from `controllers/`.
- Keep authentication behavior aligned with `definitions/auth.js`.
- Keep startup and reload behavior aligned with `definitions/init.js` and `definitions/func.js`.

## CMS Data Model

- `MAIN.id` is `db`.
- `MAIN.db` stores CMS metadata such as pages, layouts, widgets, redirects, variables, navigation, config, and storage.
- `MAIN.db.fs` is `FILESTORAGE(MAIN.id)` and stores page/layout/widget HTML content by id.
- `FUNC.load()` reads `meta` from FileStorage, initializes defaults, recompiles widgets, installs widget hooks, refreshes caches, applies config, and emits `reload`.
- `FUNC.save()` writes metadata back to FileStorage and excludes compiled widget `ref` objects.
- `FUNC.unload()` runs widget uninstall hooks, drops FileStorage state, and emits `unload`.
- Do not treat runtime FileStorage data as normal source unless the task is explicitly about backup, restore, migration, or defaults.

## Rendering Rules

- Public pages are rendered through `controllers/default.js`.
- Pages and layouts are compiled with `F.TCMS.compile()`.
- Page and layout HTML can include CMS placeholders such as `@{ui}`, `@{year}`, and `$VARIABLE` references.
- `FUNC.refresh()` rebuilds caches for widgets, dependencies, pages, and navigation.
- When page, layout, navigation, widget, or variable behavior changes, clear or refresh the affected cache and delete affected entries in `MAIN.views` when needed.
- Preserve `$.response.minify = false` behavior in public rendering unless the task explicitly changes render output.

## Plugins

- Built-in plugin logic lives under `plugins/<name>/`.
- Plugin server entry points are usually `plugins/<name>/index.js`.
- Plugin actions/schemas are usually under `plugins/<name>/schemas/`.
- Plugin UI is usually under `plugins/<name>/public/`.
- Preserve permission checks such as `permissions: 'pages,admin'`, `permissions: 'layouts,admin'`, and `UNAUTHORIZED($, 'admin')`.
- If a plugin changes public behavior, update the relevant UI files and schema/action handlers together.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [totaljs/cms](https://github.com/totaljs/cms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
