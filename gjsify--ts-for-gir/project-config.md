---
trigger: always_on
description: TS type definition generator for GObject Introspection (GIR) → GJS. Prefer retrieval-led reasoning over pre-training-led reasoning.
---

# AGENTS.md — ts-for-gir

TS type definition generator for GObject Introspection (GIR) → GJS. Prefer retrieval-led reasoning over pre-training-led reasoning.

## General

- `./types`, `./gjs`, `./vala-girs` are git submodules — **never delete**
- Always use `.ts` import extensions — TS runs directly, no build step
- Config files: `.ts-for-gir.*.rc.js` in project root

## Workspace

Yarn v4 workspaces | Node >= 22 | all ESM (`"type": "module"`)

Category | Path | Namespace | Notes
---|---|---|---
Core | `/packages/*` | `@ts-for-gir/*`, `@gi.ts/*` | No build, runs TS directly
Types | `/types/*` | `@girs/*` | Generated — **never edit manually**
Examples | `/examples/*` | `@ts-for-gir-example/*` | Req build (GJS can't run TS)
Tests | `/tests/*` | `@ts-for-gir-test/*` | Generator tests

Core packages: `cli` | `parser(@gi.ts)` | `lib` | `reporter` | `generator-base` | `generator-typescript` | `generator-html-doc` | `templates`

## Commands

```
gjsify run start                 # Run CLI (no build)
gjsify run build:{types,examples}
gjsify run copy:girs             # Copy system GIR files
gjsify run test                  # Full suite
gjsify run test:tests            # Quick local
gjsify run check                 # Full type check (SLOW!)
gjsify run check:{app,lint}      # Fast check / lint only
gjsify format                    # Biome via @gjsify/cli wrapper
gjsify run ts-for-gir-dev generate Gtk-4.0 [--reporter --verbose]
gjsify run ts-for-gir-dev analyze -f ./report.json [--severity critical --category type_resolution --namespace Gtk --format table]
gjsify run ts-for-gir-dev list
```

## Generation Flow

GIR XML (`/girs/`) → `@gi.ts/parser` → `@ts-for-gir/lib` → `@ts-for-gir/generator-typescript` → `/types/@girs/*`

### Key Files

Type improvements: `packages/generator-typescript/src/type-definition-generator.ts` | `packages/lib/src/gir-module.ts` | `packages/lib/src/transformation/*.ts`
Templates (EJS + ambient TS): `packages/templates/templates/*.d.ts` — use `<%= %>` vars, `<%- %>` unescaped
New features: extend `GeneratorBase` in `packages/generator-base/`, implement in `packages/generator-typescript/`, add templates, update types in `packages/lib/src/types/`

### Template Rules (`packages/templates/templates/*`)

- Treat as EJS + ambient TypeScript declaration context
- Preserve all EJS tags — never refactor inside `<% %>` blocks
- Ignore ESLint/TS errors in templates — they resolve in generated output
- No runtime code; ambient declarations only; minimal deterministic logic
- Stable whitespace/indentation for minimal generated diffs

### Override System (`packages/generator-typescript/src/overrides/`)

Use `node.assertClass("ClassName").noEmit()` to disable auto-generation; templates provide manual impl.

### Output Dirs

`/types/*` (submodule, branch `main`): official published types, may be cached
`/types-dev/*` (submodule, branch `dev`): development types, used by examples and workspace `@girs/*` packages
Custom `--outdir=./test-types-*`: fresh generation for dev/testing

### Validation After Changing Type Generation

When modifying generators, templates, injections, or lib code that affects generated output:

1. `gjsify run build:types` — regenerates all types into `/types-dev/`
2. `cd types-dev && git diff` — inspect generated changes, verify correctness
3. `gjsify run build:examples` — rebuild examples (they depend on `/types-dev/`)
4. `gjsify run check` — full type check including examples and generated types

`gjsify run build` chains all steps: `build:app → build:types → build:examples → build:json → build:doc`

**Important:** Examples import from `@girs/*` packages which resolve to `/types-dev/`. Generator changes will NOT be reflected in examples or `gjsify run check` until `gjsify run build:types` has been run.

### GIR → TS Mapping

`<class>` → class | `<interface>` → interface | `<enumeration>` → enum | `<signal>` → event methods | `<property>` → getters/setters | `<callback>` → function types | `<record>` → interface

Parameter `direction` affects signatures. `nullable` → optional (`?`). Array specs → array/tuple types.

## GIR Sources

`/girs/` — local copies | `/vala-girs/` — submodule | System: `/usr/share/gir-1.0/`
Add new: install pkg → `gjsify run copy:girs` → `gjsify run ts-for-gir-dev generate ModuleName-Version`

## GIR XML Reference (`**/*.gir`)

Spec: https://gitlab.gnome.org/GNOME/gobject-introspection/-/raw/main/docs/gir-1.2.rnc — impl in `packages/parser`

Namespaces: `core=".../core/1.0"` | `c=".../c/1.0"` | `glib=".../glib/1.0"` | `doc=".../doc/1.0"`

Structure: `Repository > {Include,Package,Namespace > {Class,Interface,Record,Enumeration,Function,Union,Bitfield,Callback,Constant,Alias}}`

Key attrs: `introspectable="0|1"` | `direction="in|out|inout"` | `transfer-ownership="none|container|full"` | `nullable="0|1"` | `optional="0|1"` | `glib:type-name` | `c:type`

## GVariant Types

Applies to GLib.Variant typing in `packages/templates/templates/glib-2.0.d.ts`. Uses `$ParseShallowVariant`, `$ParseDeepVariant`, `$ParseRecursiveVariant`.

Scalars: `b`→bool | `s|o|g`→str | numeric→number | `h|?`→unknown | `v`→Variant(deep)/unknown(recursive)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gjsify/ts-for-gir](https://github.com/gjsify/ts-for-gir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
