---
trigger: always_on
description: Handles `//`/`#`/`/* */` comments, strings, and PHP 8 `#[attributes]` (not treated as
---

# magequery

A fast, Rust-based developer tool for understanding a Magento 2 codebase from the command
line: modules, DI resolution (preferences/plugins/virtual types), events/observers, cron,
routes, config across scopes, and (phase 2) live DB/Redis introspection.

The value prop is **speed and zero-bootstrap**: answer "what's going on in this codebase"
in milliseconds, ideally on a checkout that has *never been set up* — no DB, no DI
compile, no working PHP. `bin/magento`/magerun bootstrap the whole framework (1–3s/call);
magequery does not.

## Locked decisions

- **Pure static engine.** Re-implement Magento's config-merge semantics in Rust by parsing
  the source XML/PHP directly. Do **not** read `var/cache` or `generated/` merged
  artifacts — those only exist after setup/compile, which breaks the fresh-checkout
  promise.
- **Target: Magento 2.4 OSS only** for the MVP. No Adobe Commerce / Mage-OS / OpenMage yet.
- **Depth-first on the flagship `resolve`/`di` command** before breadth. The DI resolver is
  the hard 80%; the other commands are simpler projections of the same index.
- **Library-first.** `magequery-core` computes and returns owned, structured data; it never
  prints, exits, or reads ambient state. The CLI is a thin renderer on top.

## Architecture

Workspace:
- `magequery-core` — parsing, indexing, resolution. Deps: `quick-xml`, `serde` (default
  feature), `thiserror`. **No `clap`, no output, no `anyhow`.**
- `magequery-cli` — `clap` + table/`--json` renderers. May use `anyhow` internally to
  flatten errors for `main`.
- `magequery-lsp` — the language server (see "LSP + editor plugins"): a third renderer
  over core, speaking LSP instead of ANSI. Depends on core **without** the `db` feature.
  (`editors/zed` holds a fourth crate, detached from the workspace — it compiles to WASM.)

### The central engine

Everything routes through one config-merge engine; subcommands are views over it.
1. **Module discovery + load order** from `app/etc/config.php` (`modules` map = enabled +
   authoritative order) and each `etc/module.xml` `<sequence>`. Load order makes merges
   deterministic.
2. **Area-aware merge.** For each config type, merge `global` (base) overlaid by the
   per-area config, in module load order.
3. **Per-node merge rules.** Preferences = last-wins, followed to a fixpoint. Plugins =
   keyed by name, honor `disabled` + `sortOrder`, split before/around/after. Observers =
   keyed by name, honor `disabled`.

### The flagship `resolve(type, area)`

```
concrete = follow_preferences(type, area)   # fixpoint over merged preference map
chain    = ancestors(concrete)              # parents + interfaces
plugins  = plugins on concrete OR any ancestor/interface, merged (global ← area),
           drop disabled, sort by sortOrder, split before/around/after
args     = merged <arguments> (+ virtualType layering + parent-type inheritance)
→ every row tagged with Source { module, file, line, area }
```

### Pure static still needs PHP parsing (not execution)

Plugins declared on an **interface or parent class apply to all implementations/
subclasses** — the case people miss. So `resolve` needs the class hierarchy, which lives
in PHP. Approach (keeps the no-bootstrap promise):
- Use composer PSR-4 autoload maps for `class → file` (pure string math; vendor is too big
  to scan — 716 modules in the test checkout).
- Parse PHP **on demand**, only for classes on the resolution path, extracting just
  `extends`/`implements` from the class header. Cache it. (tree-sitter-php or a focused
  header parser — never execute PHP.)

### Areas

Fixed 2.4 OSS set, hardcoded, never discovered from the filesystem (`etc/` contains
non-area dirs like `postcode_eu/`, `some_config/`, `redis/`):
`global, frontend, adminhtml, crontab, webapi_rest, webapi_soap, graphql`. `global` is the
base; every real area = `global` overlaid by itself.

CLI area model:
- *(default)* collapsed diff — `global` base + per-area deltas only (silence = same
  everywhere = information).
- `--area <name>` — single area.
- `--all-areas` — full per-area expansion. (`--area`/`--all-areas` are mutually exclusive.)
- `routes` defaults to all-areas (frontend vs adminhtml routers are the point).

The collapse lives in **core** as `ByArea::deltas()`, so library users and the CLI render
from the same computation.

## Type design (`magequery-core`)

- **Typed identifiers, never stringly-typed**: `ClassName`, `ModuleName`, `EventName`,
  `ConfigPath`, `Area` (enum). `ClassName::new` **strips a leading backslash** (`\Foo\Bar`
  ≡ `Foo\Bar`) — the invariant is enforced at construction, not at call sites, mirroring
  Magento's `ltrim($type, '\\')` at every config read. Both spellings occur in real di.xml
  (core module-elasticsearch writes `type="\Magento\…"`) and must merge/compare as one
  name; before this was enforced, `uses` missed backslash-declared virtualTypes and their
  arg inheritance silently failed to merge.
- **Provenance everywhere**: `Source { module, file, line, area }` on every returned fact;
  `.location()` → clickable `file:42`. This is the whole point — answers jump to source.
- **Errors vs diagnostics split** (the key, hard-to-retrofit decision):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cresset-tools/magequery](https://github.com/cresset-tools/magequery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
