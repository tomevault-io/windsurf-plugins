---
trigger: always_on
description: Before any code or documentation edit, read:
---

# Qx Project — Agent Guidelines

## Read First

Before any code or documentation edit, read:

1. `UI_SPEC.md` — current UI, theme, layout, interaction, and validation rules.
2. `TASK.md` — current project tasks and known verification status.
3. `AGENTS.md` — this operating guide.
4. `docs/architecture-principles.md` — SOLID, abstraction layers, interface contracts, doc duty.
5. For **global shortcuts, panel show/hide, or Tauri `State` / `.manage()`**: `docs/shell-and-shortcuts.md`.
6. For **new built-in modules, shell/Esc/list ports, or marketplace plugins**:
   `docs/module-port-inventory.md` + `public/doc/plugin-development-guide.md`.

If the request is UI-related, treat `UI_SPEC.md` as the source of truth. Do not invent alternate layout systems or component conventions.
If the request is **QxAI chat / workbench / message / composer / queue / token-speed** visual or interaction, also treat [`UI_SPEC_AI.md`](UI_SPEC_AI.md) as the source of truth: **AI Elements structure** + **Beautiful UI visuals** (not a full SDK install). Shell chrome still follows `UI_SPEC.md`.
If the request changes **public interfaces or layer boundaries**, update docs in the same change.

## Working Rules

- Preserve user or concurrent changes. Never revert unrelated dirty files.
- Prefer existing patterns and local helper APIs over **new** abstractions.
  When a new abstraction *is* required, design it as a narrow, stable port and
  document it — see **Architecture Principles (SOLID)** below.
- Keep edits scoped to the request.
- **Maintain docs with the code.** Public interfaces, RPC/commands, permissions,
  and layer boundaries must update the matching file under `docs/` or
  `public/doc/` in the same change. Prefer intent and invariants over dumping
  implementation detail.
- **Do not fix call sites one-by-one** when a port is wrong (host HTTP, i18n
  dictionary, shell Esc, island session). Fix the port once, migrate all
  first-party consumers to the corrected Qx protocol, then run `npm run check`.
- Before finishing a multi-file change: `npm run check` (architecture + docs +
  i18n + shell + island + module-ports gates).
- Use `rg` / `rg --files` for search.
- Use `apply_patch` for manual edits.
- Do not introduce generated build artifacts, secrets, temp files, or unrelated formatting churn.

## Documentation and build-output boundaries

Keep the three documentation/build locations separate; they are not alternate
copies of the same source:

- `docs/` contains internal contributor and architecture documentation (ports,
  threading, storage, shell contracts, and design decisions).
- `docs/README.md` and `public/doc/README.md` intentionally have the same
  basename but are different indexes: the former is for core contributors and
  the latter is for plugin authors, operators, and release maintainers. The
  matching filename is not a duplicate to remove.
- `public/doc/` contains user-, operator-, and plugin-author-facing Markdown.
  This is the canonical source for documents that Qx links from its UI or
  README, including the plugin development and release guides. Edit this tree,
  not its build output.
- `dist/` is generated Vite output. `dist/doc/` is only the build-time copy of
  `public/doc/`, alongside compiled JavaScript, CSS, and assets. Never edit or
  commit anything under `dist/`; remove it when stale and regenerate it with
  `npm run build`. A root-level `doc/` directory is not part of the project and
  must not be created as a third documentation source.

When the same Markdown appears under `public/doc/` and `dist/doc/`, the
`public/doc/` file is authoritative and the `dist/doc/` copy is disposable
build output. Do not maintain both versions or manually “fix” the generated
copy. If a duplicate source document is found elsewhere, consolidate it into
the appropriate canonical tree and update links in the same change.

## Architecture Principles (SOLID)

Full write-up: [`docs/architecture-principles.md`](docs/architecture-principles.md).

Qx interfaces and modules must stay **abstract enough to extend**, without
becoming vague. Apply SOLID at the port boundary:

| Letter | In this repo |
|---|---|
| **S** | One reason to change per module (`QxShell` = chrome; feature view = domain UI; Rust module = domain service). |
| **O** | Extend via registration / adapters (builtin catalog, island modes, host capabilities) — do not grow core `switch` forests for every feature. |
| **L** | Same command / context / session shape on every platform and for real vs unavailable plugin contexts. |
| **I** | Narrow surfaces: capability permissions, focused host APIs, per-package shims — no God context. |
| **D** | Features depend on stable ports (`invoke`, plugin context, island hostApi, `useT`); OS and iframe details stay below the port. |

The Raycast converter is frozen and retained only for historical experiments.
For maintained plugins, read the upstream source and reimplement its business
intent against Qx `context.*`, Workbench, Actions, and Island protocols. When a
shared capability is missing, fix the host port once and migrate every
first-party plugin; do not extend converter shims as the production path.

### Module Decomposition (required)

- Decompose by domain boundary and reason to change, not by line count alone.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mcxen/qx](https://github.com/mcxen/qx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
