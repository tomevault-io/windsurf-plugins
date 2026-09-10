---
trigger: always_on
description: The build contract for lottie-react. It applies to every change in this repository, whether written by a person or an agent. Read it before the first edit.
---

# CLAUDE.md

The build contract for lottie-react. It applies to every change in this repository, whether written by a person or an agent. Read it before the first edit.

## The project

lottie-react is a React wrapper around [lottie-web](https://github.com/airbnb/lottie-web) for rendering After Effects animations. It ships a component (`Lottie`), a hook (`useLottie`), the same pair twice more on lottie-web's smaller builds (`LottieSvg` and `useLottieSvg` on `lottie_svg`, `LottieLight` and `useLottieLight` on `lottie_light`), a player UI, and an interactivity layer.

v3 is the released line at `3.0.0`; `main` carries it, and the `v2` branch holds the v2 line for fixes.

## Hard rules

1. **The public API is a contract.** Anything exported from `src/index.ts` is depended on by a large number of projects. Within a major version, nothing exported may change shape, change meaning, or disappear. v3 is the major where breaks are allowed, and each one ships with a migration note in the same change.

2. **Server rendering must never break.** No access to `document`, `window`, or any browser global at module scope, during render, or in any code path a server can reach. Browser work belongs in effects and event handlers. When it is unclear whether a path runs on the server, the safe resolution is to not touch the DOM.

3. **Never implement against remembered lottie-web behavior.** lottie-web is the moving dependency underneath this library, and its behavior differs across versions and renderers. Check the installed package's types, source, and changelog before relying on a method, option, or event. The same applies to React version differences and to bundler resolution behavior.

4. **Stay small.** Nothing here ships as one blob. The build emits a file per module and the package declares `sideEffects: false`, so the size that matters is per import rather than per package. The numbers live in one place: the `size-limit` budgets in `package.json`, measured with React and the engine left external, minified and gzipped, and `pnpm check` fails when an import outgrows its budget. Growth is therefore a reviewed raise of a budget line, never a drift, and it stays a cost that has to be justified: keep new runtime dependencies close to zero, and never let the svg or light entry points reach any engine but their own. That last one is measured rather than trusted: the two budget lines naming those pairs leave only the pair's own engine external, so any other engine reaching them lands in the bundle and fails the budget by an order of magnitude.

   **`sideEffects: false` is a promise, and the build is shaped around it.** It tells a consumer's bundler that importing a file here does nothing by itself, so any file whose exports go unused may be deleted whole. `unbundle` in `tsdown.config.ts` is what gives it something to delete, because a single merged file is always in use, so there is nothing to drop. A bundler acts on the field without verifying it, so declaring it falsely removes code that should run, with no error and usually no symptom until production. It stops being true the moment a file is imported for its effect (`import "./styles.css"`), does work at module scope, or discards the result of a module-scope call. This library's CSS is rendered by React during render rather than imported, which is what keeps the claim honest. If a real stylesheet ever ships, narrow the field to the array form rather than removing it.

5. **React 18 and 19 are both supported.** The peer range promises both, so a change that only works on one is not finished.

6. **The gate decides done.** `pnpm check` passes before any change is called finished. A red gate is the verdict, not a starting point for an opinion about whether the change was fine anyway.

## Conventions

These describe what the code already does. Follow them so the codebase stays uniform.

### Types and values

No native `enum`. A fixed set of values is an `as const` object map with a derived union type of the same name, which gives the same ergonomics and exhaustiveness without the runtime and erasability problems. `src/types/enums.ts` is the template:

```ts
export const Direction = {
  right: "right",
  left: "left",
} as const;
export type Direction = (typeof Direction)[keyof typeof Direction];
```

No non-null assertions (`!`) and no `any`. Neither appears in `src` today. Narrow with a guard, or make the type honest.

Use `interface XProps` for object-shaped props and `type` for unions and aliases.

### Modules and exports

Named exports throughout. `src/index.ts` is the single public barrel, and a symbol is public only if it is exported there. Default exports are limited to small single-purpose utilities and hooks.

A file's name matches the symbol it exports, in the same casing. Components are PascalCase, hooks are `useThing.ts`, utilities are camelCase. Casing is not cosmetic here: a mismatch between a filename and its import specifier works on macOS and fails on a case-sensitive filesystem.

Import order is enforced by Biome's `organizeImports` assist and is auto-fixable with `pnpm format`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Gamote/lottie-react](https://github.com/Gamote/lottie-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
