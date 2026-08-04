---
trigger: always_on
description: **Purpose:** a TypeScript UI framework where Effect's `<A, E, R>`
---

# verrex — Effect-native UI framework

**Purpose:** a TypeScript UI framework where Effect's `<A, E, R>`
channels propagate from every leaf of the view tree to the root.
Forgetting to provide a service `Layer` becomes a _compile-time
error that names the missing service_; symmetrically, forgetting to
handle an error with a `Catch` boundary becomes a _compile-time
error that names the unhandled error_ (`mount` requires
`Effect<View<never>, never, R>`). Errors live in two phases: construction
errors ride the Effect `E`; live errors a rendered subtree can still
produce ride the `View<E>` success — one `Catch` boundary discharges both.

**Honest scope today:** the _construction_ channel is fully type-tracked —
a forgotten boundary on a failing build is a compile error. The _live_
channel is tracked at two leaves. (1) `Async` _without_ a `failure` arm (or
with a _partial_ tag map, whose residual rides), typed
`Effect<View<E>, never, R | Scope>`. Its failures (initial fetch or refetch)
ride `View<E>` to the nearest `Catch`, and `mount`'s `View<never>` gate makes
a missing boundary a compile error naming `E`. The `failure` arm mirrors
`Catch`'s two forms: a function handles everything at the leaf
(`View<never>`); a tag map handles matched tags at the leaf — keeping the
fetch loop live, so a dep change can recover the view — while the residual
rides `View<Exclude<E, { _tag }>>`. (2) _Event handlers_ (#72): an intrinsic's
`on*` prop returning `Effect<_, E, R>` stamps `E` on the element's `View<E>`
and folds `R` into its requirements — a forgotten boundary or Layer for a
handler is the same compile error as for construction. The remaining
untracked live surface: a _reactive re-render_ whose Effect fails (an
`AtomRef`-driven child re-emitting a failing Effect) is still caught only at
_runtime_ by `Catch`'s sink, not typed.

**The name** is built from the channels of an `Effect<View, E, R>`:
**V** (View — the `A`, always the `View` here), **E** (Error), **R**
(Requirements), plus **X**, because the JSX/TSX syntax it borrows adds an
X too. `V + E + R + X` spells **verx**, stylized to **verrex** (and the
`.vx` source extension).

Status: experimental proof-of-concept. Not production-ready.

## The constraint that shaped everything

TypeScript's JSX type-checker erases generic type variables at the
JSX boundary — every component result collapses to `JSX.Element`.
React, Solid, Preact all live with this. For a framework where
the _point_ is that `E`/`R` channels survive composition, that's
fatal.

So this project deliberately **never lets `tsc` see JSX**:

1. Source files use a custom `.vx` extension.
2. `@verrex/core/compiler` (Babel-based) rewrites every JSX node into an
   `h(tag, props, ...children)` call **before** tsc sees the file.
3. `h()`'s generic signature in `verrex` uses conditional
   types (`FoldE`/`FoldR`) to union every child's `E` and `R` into
   the result `Effect<View, E, R>`.

Everything else — the `.vx` extension, the Babel choice, the
custom Vite dev server hook, the TS Language Service plugin —
exists to support this constraint.

### "JSX" here means JSX _syntax_, not JSX _semantics_

This is the most important framing for anyone (or any agent)
joining the codebase. **We borrow JSX syntax. We do not use JSX
in any other sense.**

We use:

- The angle-bracket form `<div>...</div>` as a source-code shape.
- Babel's `jsx` parser plugin to recognize that shape.
- Editor / Treesitter JSX highlighting (via `typescriptreact`
  filetype mapping).

We do **not** use:

- A JSX runtime (`jsx-runtime`, `jsx-dev-runtime`,
  `React.createElement`, none of it).
- The `JSX` TypeScript namespace (`JSX.IntrinsicElements`,
  `JSX.Element`). If those names appear in an error message,
  something is wrong — tsc is seeing JSX it shouldn't.
- Any React-shaped library. There is no React, Preact, Solid
  dependency.
- TypeScript's JSX type-checker. It's never engaged because the
  compiler removes the syntax before tsc parses the file.

Post-compile, `Counter.vx` is `h("div", { class: "counter" }, ...)`
calls in a `.ts` file. Plain function calls in plain TypeScript.
That's the only thing tsc, Vite, your IDE's type-checker, or any
downstream tool ever sees.

When this AGENTS layer says "JSX expression," "JSX node," or "JSX
child," read it as "the angle-bracket source-code shape that the
compiler eats and converts into `h()` calls." Not the React thing.

## Subsystems

Everything user-facing ships as one package, **`packages/core/`**, with a
subpath export per surface (the subdirs self-reference via `verrex/*`). The
editor plugin is the one separate package, because tsserver resolves Language
Service plugins only by bare package name.

- **[`src/runtime/`](./packages/core/src/runtime/AGENTS.md)** — export `@verrex/core`. `h`,
  `mount`, `Async`, `asyncRef` (returning `AsyncHandle`), `list`, `Catch`, the View IR (mount switches on it),
  reactivity wiring, channel-fold types. The thing components import from.
- **[`src/compiler/`](./packages/core/src/compiler/AGENTS.md)** — export
  `@verrex/core/compiler`. The Babel transform: intrinsic JSX → `h()`,
  component tags → direct calls (`MyComp({...})`), `.value` → `h.read()`,
  `<expr>.value.map(arrow → JSX)` → `list(<expr>, arrow)`. Smart-skip wrap.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [m9tdev/verrex](https://github.com/m9tdev/verrex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
