---
trigger: always_on
description: <!--VITE PLUS START-->
---

<!--VITE PLUS START-->

# Using Vite+, the Unified Toolchain for the Web

This project is using Vite+, a unified toolchain built on top of Vite, Rolldown, Vitest, tsdown, Oxlint, Oxfmt, and Vite Task. Vite+ wraps runtime management, package management, and frontend tooling in a single global CLI called `vp`. Vite+ is distinct from Vite, and it invokes Vite through `vp dev` and `vp build`. Run `vp help` to print a list of commands and `vp <command> --help` for information about a specific command.

Docs are local at `node_modules/vite-plus/docs` or online at https://viteplus.dev/guide/.

## Built-in Commands vs Scripts

`vp <name>` runs a built-in command. `vp run <name>` runs a `package.json` script or a `vite.config.ts` task. Scripts cannot overwrite built-ins, so `vp dev` and `vp run dev` may do different things. Check `package.json` and `vite.config.ts` first, and run `vp run <name>` when the project defines a script or task with that name.

## Tool Versions

Run `vp toolchain` to show versions and relationships in the active Vite+
release. Add a tool name to select part of the graph. For example, run
`vp toolchain vite`. Use `--global` to ignore the local `vite-plus` package. Use
`vp why <package>` to show the package-manager dependency graph.

## Review Checklist

- [ ] Run `vp install` after pulling remote changes and before getting started.
- [ ] Run `vp check` and `vp test` to format, lint, type check and test changes.
- [ ] Check if there are `vite.config.ts` tasks or `package.json` scripts necessary for validation, run via `vp run <script>`.
- [ ] If setup, runtime, or package-manager behavior looks wrong, run `vp env doctor` and include its output when asking for help.

<!--VITE PLUS END-->

# Vuestrata Project Rules

Everything below is project-specific and is not managed by Vite+.

**This file is the canonical brief.** `.github/copilot-instructions.md` and
`CLAUDE.md` point here rather than repeating it, so there is one place to change
when a rule changes.

Highest authority above this file: `.specify/memory/constitution.md`. Read it
before major implementation, and flag conflicts rather than silently violating
them.

## Who owns which file

Projects built from this template take releases through `vuestrata upgrade`,
which can only work because every file the tooling writes belongs to exactly one
side. Before editing anything under `src/modules/app/` or `src/modules/core/`,
know which of these it is.

| Class     | Meaning                                   | On upgrade                                     |
| --------- | ----------------------------------------- | ---------------------------------------------- |
| `managed` | Vuestrata's                               | Replaced, unless the project edited it         |
| `seeded`  | Written once, then the project's          | Never touched again                            |
| `merged`  | The project's, except one anchored region | Only the region between the markers is written |

`packages/cli/src/lib/managed.mjs` is the list. In short: the `Ui*` layer,
`composables/forms/`, `styles/themes/`, `core/lib/` and the layout components
are `managed`; `brand.css`, `app.overrides.ts`, `Logo.vue` and the locale
overrides are `seeded`; the registries are `merged`.

**Every registry carries two sentinel regions**, and they must not be merged
back into one:

```ts
// vuestrata:modules-start   an upgrade writes here
// vuestrata:modules-end
// app:modules-start         generators write here; upstream never does
// app:modules-end
```

Generators write to `app:`. If you are adding something Vuestrata ships, it goes
in `vuestrata:`. `vuestrata doctor` reports any region that has gone missing.

**Do not edit a `managed` file to change something a seam already covers.** A
colour belongs in `brand.css`, a string in `<locale>.overrides.json`, the product
name and footer links in `app.overrides.ts`. Editing the underlying file works,
and costs that file its updates forever.

Breaking any of the contracts in `RELEASE.md` requires a migration under
`packages/cli/migrations/<version>/`. That is a release rule, not a preference.

## Extending the template

Most extension tasks have a generator. Each writes the files **and** the
registries that are easy to forget, then formats what it wrote.

| Task               | Command                        | Recipe                                 |
| ------------------ | ------------------------------ | -------------------------------------- |
| CRUD domain module | `vpr gen:module <name>`        | `docs/9.recipes/1.add-a-module.md`     |
| Page in a module   | `vpr gen:page <module> <name>` | `docs/9.recipes/2.add-a-page.md`       |
| Theme              | `vpr gen:theme <name>`         | `docs/9.recipes/3.add-a-theme.md`      |
| `Ui*` component    | `vpr gen:component <Name>`     | `docs/9.recipes/4.add-a-component.md`  |
| Icon provider      | `vpr gen:icon-set <name>`      | `docs/9.recipes/5.add-an-icon-set.md`  |
| Locale             | — by hand                      | `docs/9.recipes/6.add-a-locale.md`     |
| Permission         | — by hand                      | `docs/9.recipes/7.add-a-permission.md` |
| Nav group          | — by hand                      | `docs/9.recipes/8.add-a-nav-group.md`  |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [boussadjra/vuestrata](https://github.com/boussadjra/vuestrata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
