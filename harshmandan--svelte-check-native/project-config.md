---
trigger: always_on
description: This file is loaded into every Claude Code session in this repo. Read
---

# Working conventions for Claude Code / AI-assisted contributions

This file is loaded into every Claude Code session in this repo. Read
`README.md` for the public-facing overview, and `ROADMAP.md` (local,
gitignored) for the current scoreboard, what's next, and documented
limitations. This file is the shorter "rules of engagement" layer on
top of both.

## Project at a glance

- **Goal:** a CLI-only type checker for Svelte projects, written in
  Rust, powered by tsgo. Drop-in replacement for upstream `svelte-check`
  on the CLI surface (same flags, same output formats, same exit codes,
  same `<N> FILES` denominator).
- **Svelte 4 and Svelte 5 are both supported.** Svelte-4 surface
  features (`export let`, `$:`, `on:event`, `<slot>` / named slots,
  `createEventDispatcher`, `bind:` on components, renamed exports)
  all shipped in the v0.2 parity push. Parity gate: a 1000-file
  mid-migration SvelteKit workspace type-checks clean, tying upstream
  `svelte-check --tsgo` at 0 real errors.
- **No bundled tsgo.** We discover the user's `@typescript/native-preview`
  install in `node_modules`, preferring the platform-native binary over
  the JS wrapper. `TSGO_BIN` env var is the override.
- **Upstream submodule:** `language-tools/` is a pinned submodule of
  `sveltejs/language-tools` — used as the source of truth for upstream's
  CLI behavior, the 63 `.v5` test fixtures from `svelte2tsx` that form
  our parity gate, and the `isKitFile` / `findFiles` algorithms whose
  output we mirror byte-for-byte.

## Scope discipline (repeated here because it's easy to forget)

Out of scope — do NOT implement:

- LSP server / editor integration
- Autocomplete, hover, go-to-definition, rename, code actions
- Watch mode (use `watchexec` externally)
- tsc fallback (tsgo only)
- Formatting
- CSS lint rules beyond the narrow vendor-prefix carve-out in ROADMAP

Svelte-4 compat is shipped but isolated: every Svelte-4-specific
helper goes under `crates/*/src/svelte4/` with a `// SVELTE-4-COMPAT`
marker at each callsite. When Svelte 4 is officially retired the
removal is mechanical — delete the submodule and grep for the marker.

### Svelte-4 feature coverage (internal reference)

What each Svelte-4-specific surface feature maps to in the emit:

| Syntax                                 | Handled by                                  |
| :------------------------------------- | :------------------------------------------ |
| `export let foo` / `export let foo: T` | `crates/analyze/src/props.rs` — synthesized Props entry |
| `export let foo = v` (untyped default) | widen to `any` in script_split so Props stays permissive |
| `export { name as alias }`             | specifier-form in `crates/analyze/src/props.rs` |
| `$: NAME = EXPR` (declaration)         | `crates/emit/src/svelte4/reactive.rs` — `let NAME = __svn_invalidate(() => EXPR); void NAME;` |
| `$: EXPR;` (statement)                 | arrow-wrap: `;() => { $: EXPR };` so body's TDZ is function-scope not top-level |
| `$: ({a, b} = expr)` (destructure)     | rewrite to `let {a, b} = __svn_invalidate(() => (expr));` |
| `on:event={handler}` on components     | `ComponentInstantiation.on_events` + emit `$inst.$on("evt", handler);` after construction |
| `on:event={handler}` on DOM elements   | translated to `onXXX` attribute key in `svelteHTML.createElement` call |
| `<slot>` / `<slot name="X">` / `<slot {prop}>` | synthesized `children` / named Snippet props on the generated Props type |
| `<Foo let:item>` slot-let subtree      | let-binding scope opened before `emit_component_call`; `let item: any; void item;` |
| `createEventDispatcher<T>()`           | dispatcher call recognized; `$on` signature keeps `handler: (...args: any[]) => any` (matches upstream's `__sveltets_2_with_any_event` widening of the default export) |
| `bind:` on components                  | silently dropped at analyze; `__SvnPropsPartial<P>` makes the target prop optional so nothing fires |
| `bind:this={x}` on elements            | `BindThisTarget` recorded; script's `let x: T` gets the `!` definite-assign rewrite so async bind doesn't trip TDZ |
| `$$Props` / `$$Events` / `$$Slots`     | `crates/analyze/src/svelte4/` — pulls the interface as the props-type source if present |
| `$$slots` / `$$props` / `$$restProps` ambients | injected per-file in the script prelude so references resolve |
| `export function` / `export const`     | surfaced on the synthesized instance return; consumers see the field via `__SvnInstance<P>` + default-export type |

Parity gate: 0 errors on a 1000-file mid-migration SvelteKit
monorepo, tying upstream `svelte-check --tsgo`. See `ROADMAP.md`
for the exact bench numbers and known tsgo ceiling limits.

In scope: CLI flags matching upstream, byte-identical output formats,
tsgo invocation, diagnostics mapping back to `.svelte` source,
`<N> FILES` denominator matching upstream's `|entries ∪ diagnostic
files|` semantic.

## Commit-and-continue

- **Commit after every meaningful local step,** even if code is broken or
  tests are red. Commits are restore points, not polished artifacts.
- **Never `git push` without explicit user confirmation** each time.
  Session-level approval does not carry over to future sessions or
  branches.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harshmandan/svelte-check-native](https://github.com/harshmandan/svelte-check-native) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
