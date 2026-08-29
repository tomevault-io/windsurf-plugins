---
trigger: always_on
description: This is a monorepo for a component library built on real semantic HTML.
---

# Working in this repo

This is a monorepo for a component library built on real semantic HTML.
Read `README.md` for the project pitch. This file is operational
guidance for anyone (human or agent) making changes here.

## Release checklist for pull requests

Any PR that changes a publishable package under `packages/react`,
`packages/elements`, `packages/styles`, or `packages/cli` must include a
`.changeset/<descriptive-name>.md` file describing the affected package(s) and
the release level (`patch`, `minor`, or `major`). Run `bunx changeset` to create
it. Docs-only changes do not need a Changeset. CI enforces this rule; the only
exception is the Changesets-generated `Version Packages` release PR.

When creating a PR, never leave the Changeset decision implicit: add the file
or explicitly confirm that the PR is docs-only/non-publishable. A merged PR
without a Changeset will not produce an npm or GitHub release.

For **consuming** the published packages, don't read this file — read
`packages/react/llms.txt` or `packages/elements/llms.txt` instead; they're
the API reference. This file is about *building* the library, not using it.

## The one rule that matters most: build every component twice

Every component ships in **both** `@kernelui-lib/react` (`packages/react/src/components/<Name>/`)
and `@kernelui-lib/elements` (`packages/elements/src/components/<Name>/`) — a
React version and a framework-free Custom Element version, with matching
visual design and equivalent props/attributes. Adding a component to only
one package is an incomplete PR, not a smaller one. Before considering any
new component done, grep both `packages/react/src/index.ts` and
`packages/elements/src/index.ts` for its export.

## Repo layout

- `packages/react` — `@kernelui-lib/react`. Each component: `<Name>.tsx` +
  `<Name>.module.css`, exported from `src/index.ts` in shipping order
  (not alphabetical — new exports go at the end, before the `polymorphic`
  utils export block).
- `packages/elements` — `@kernelui-lib/elements`. Each component: `<Name>.ts`
  (extends `KernelElement` from `../../base`, registers via
  `customElements.define`) + `<Name>.css`, exported from `src/index.ts`.
  CSS class names use `kernelClass("Name", "part")`, which produces the
  **exact same class names** React's CSS Modules compile to (see
  `packages/react/vite.config.ts`'s `generateScopedName`) — this is
  deliberate, so one set of component styles serves both packages.
- `packages/styles` — `@kernelui-lib/styles`. Design tokens
  (`tokens.css`) and reset (`reset.css`). Both packages' components read
  tokens from here; never hardcode a color/spacing/radius value in a
  component's own CSS.
- `apps/docs` — Astro docs site. Each component needs: an entry in
  `packages/registry` (re-exported by `apps/docs/src/data/components.ts`),
  a page at `src/pages/components/<slug>.astro`, and
  `src/components/demos/<Name>Demo.tsx` + `<Name>Playground.tsx`. Copy the
  structure of an existing page (`text-field.astro`/`scroll-area.astro` are
  good recent examples) rather than inventing a new layout.
- `packages/registry` — shared component catalog consumed by docs, CLI, and
  LLM asset generation. Run `bun run build` there after changing entries.
- `packages/cli` — `@kernelui-lib/cli`. Published integration CLI (`kernel init`,
  `kernel doctor`, `kernel docs`).

## Conventions to match, not reinvent

- Controlled/uncontrolled state: `value`/`defaultValue`/`onValueChange`
  via `useControllableState` (`packages/react/src/utils/useControllableState.ts`).
- `label`/`hideLabel`/`description`/`errorMessage`/`invalid`/`disabled`
  scaffold for form fields — copy `TextField.tsx`'s structure.
  `hideLabel` visually hides the label via the shared `kernel-sr-only`
  utility class (`packages/styles/src/reset.css`) — never actually drop
  `label` from the DOM; an unlabeled input is an accessibility bug, not
  a simplified one.
- `className`/`wrapperClassName` + `resolveClassName`/`dataAttr`/`mergeRefs`
  from `packages/react/src/utils/polymorphic.ts`.
- Real elements over ARIA-only patterns wherever a native element exists.
  Every component's top JSDoc comment explains *why* it's built the way
  it is (which native element, which ARIA pattern, what tradeoff) — write
  one for every new component; this is a strict, load-bearing convention,
  not decoration (it's also what agents read via package `llms.txt` and the
  docs site's generated markdown mirrors).
- Icons are hand-authored inline `<svg>` with `stroke="currentColor"`
  (see `Toast.tsx`'s close icon or `Checkbox.tsx`'s checkmark) — no icon
  library dependency in either package.
- **Shape baseline — a radius is never picked on its own.** Every corner
  radius in the library has a padding it must be paired with, because
  `--kernel-radius-container`/`-sheet` are *derived from* the padding
  tokens (`radius-md + padding-*`), so the radius grows with a consumer's
  `--kernel-radius-base` while a hand-picked `var(--kernel-space-3)`
  doesn't. That's the drift: it looks fine at the default radius and reads
  as cramped text jammed into a giant curve at Round. The pairings, all
  three mandatory:

  | `border-radius` | padding it must use |
  | --- | --- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kemiljk/kernel-ui](https://github.com/kemiljk/kernel-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
