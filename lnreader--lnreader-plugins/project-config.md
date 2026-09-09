---
trigger: always_on
description: This repository contains the community-maintained source plugins for LNReader. Most changes are
---

# AGENTS.md

## Repository overview

This repository contains the community-maintained source plugins for LNReader. Most changes are
one of the following:

- a standalone TypeScript plugin in `plugins/<language>/`;
- a source definition or shared template under `plugins/multisrc/`;
- a plugin icon or custom asset under `public/static/`;
- the React/Vite plugin playground under `src/`; or
- build, publishing, and live-check tooling under `scripts/`.

Read `README.md` for the project entry points, `docs/quickstart.md` before adding a plugin, and
`docs/docs.md` for the plugin API. `docs/testing.md` explains the live-site checker.

## Environment and package management

- Use Node.js 22 or newer. CI currently exercises Node.js 20 and 24 depending on the workflow, so
  avoid APIs unavailable in those versions when editing tooling.
- Use npm for repository commands. The documented setup is `npm install`, and CI installs with
  `npm ci`.
- Do not update dependency lockfiles unless the task changes dependencies. If dependencies do
  change, keep `package-lock.json` consistent with `package.json`; do not rewrite lockfiles merely
  because another package manager is available.
- Copy `.env.template` to `.env` only when local manifest serving needs a custom content base.
  Never commit `.env` or credentials.

## Repository map

- `plugins/<language>/*.ts`: hand-authored plugins, grouped by the full language name.
- `plugins/multi/`: plugins that are intrinsically multi-language or server-based.
- `plugins/multisrc/<generator>/`: generator, template, source metadata, and optional generator
  documentation for families of sites using the same CMS/theme.
- `public/static/src/<language-code>/<plugin-id>/`: icons and optional plugin assets. These folders
  use short language codes such as `en`, unlike `plugins/<language>/`.
- `src/types/plugin.ts`: canonical plugin interfaces.
- `src/libs/`: runtime-compatible helpers available to plugins through the `@libs/*` alias.
- `src/`: the React/Vite playground used for interactive testing.
- `scripts/`: compilation, manifest, publishing, icon, site, and live-check tooling.
- `docs/plugin-template.ts`: starting point for a standalone plugin.
- `BLACKLIST.json`: sites that must not be reintroduced as plugins.

## Plugin implementation rules

- Before adding a plugin, check `plugins/multisrc/` for a matching site theme. Prefer adding a
  `sources.json` entry to an existing generator over duplicating its parser in a standalone file.
- Put standalone plugins in the folder matching the novels' language, using a `.ts` extension.
- Implement `Plugin.PluginBase` and export one instantiated plugin as the default export. Keep the
  plugin `id` unique and stable.
- Use imports from `@libs/*` for plugin runtime helpers. In particular, use `@libs/fetch`, not
  `@/lib/fetch`. Consult `docs/plugin-template.ts` and nearby plugins for supported helpers.
- Plugins are compiled for an ES5/Hermes/React Native environment. Do not assume Node-only or
  browser-only globals and APIs are available in the app runtime.
- Use semantic versions. Increment the version whenever modifying an existing plugin: patch for a
  small compatibility fix, minor for an improvement or feature, and major for a breaking change
  such as replacing the site/domain behavior.
- Add a 96x96 PNG icon at `public/static/src/<short-language-code>/<plugin-id>/icon.png` and set
  `icon` to `src/<short-language-code>/<plugin-id>/icon.png` (without `public/static/`).
- Keep returned paths and `resolveUrl` behavior consistent. Exercise pagination, filters, chapter
  ordering, covers, status, summaries, and chapter cleanup when the target site exposes them.
- Use `defaultCover` when a source has no usable image. Do not invent metadata that the source does
  not provide.
- A `*.broken.ts` suffix deliberately excludes an unavailable plugin from normal production
  compilation while retaining its source. Do not add or remove that suffix without confirming the
  site's current behavior and adjusting the plugin version when it returns to service.
- Check `BLACKLIST.json` before introducing a new source.

## Multi-source plugins

- Treat files named like `plugins/<language>/<name>[<generator>].ts` as generated output. Do not
  edit or commit them; they are gitignored and ESLint intentionally excludes them.
- Change the generator's `sources.json`, `template.ts`, filters, custom assets, or `generator.js`
  instead. Follow a generator-specific `README.md` when present because metadata fields differ.
- Update the generator's version increment field when its source URL or behavior changes, according
  to that generator's conventions.
- Run `npm run build:multisrc` after a generator change and inspect the generated plugin locally.
  Generated files are disposable and may be removed with `npm run clean:multisrc`.

## Coding style

- Follow the repository Prettier configuration: two spaces, single quotes, trailing commas, and no
  parentheses around a single arrow-function parameter.
- Prefer `type` aliases over `interface`; ESLint enforces this convention.
- Keep changes focused. Reuse existing parsing helpers and patterns from a nearby plugin or the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LNReader/lnreader-plugins](https://github.com/LNReader/lnreader-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
