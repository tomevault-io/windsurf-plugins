---
trigger: always_on
description: Detailed documentation lives in `.claude/skills/project-knowledge/references/`:
---

# Radix NG Primitives — Claude Guide

## Project knowledge

Detailed documentation lives in `.claude/skills/project-knowledge/references/`:

- **`project.md`** — what the library is, audience, available primitives
- **`architecture.md`** — tech stack, monorepo layout, composition primitives, DI context, CDK migration
- **`patterns.md`** — naming, new primitive checklist, signals conventions, testing, story patterns
- **`deployment.md`** — CI/CD, release process, Storybook/docs builds
- **`ux-guidelines.md`** — demo styling system, semantic tokens, animation patterns

Read the relevant file before starting any non-trivial task. The sections below in this file are a quick-reference summary; the `references/` files are the authoritative source.

## Project overview

Angular port of headless UI primitives. This is a **signals-first, headless** component library — directives carry no styles; state is exposed via `data-*` attributes for consumers to style.

**Reference priority:** [Base UI](https://base-ui.com/) is the primary reference we align to — match its component APIs, naming, and behavior when designing or updating primitives. [Radix UI](https://www.radix-ui.com/) was the original foundation of this library and remains a valid secondary reference to consult, especially for patterns Base UI doesn't cover.

- **Monorepo**: Nx 22, pnpm workspaces
- **Angular**: 21 / 22 (peer range `^21.0.0 || ^22.0.0`; signals API: `input()`, `model()`, `computed()`, `signal()`, `linkedSignal()`)
- **TypeScript**: 5.9
- **Testing**: Vitest + AnalogJS Angular Vite plugin + `@testing-library/angular`
- **Storybook**: 10 (`@storybook/angular` + AnalogJS vite plugin)
- **Styling in stories**: Tailwind v4 (see "Stories & Storybook")
- **Prefix**: `rdx` (selectors and `exportAs`)
- **Class prefix**: `Rdx`

## Monorepo structure

```
packages/
  primitives/          ← main Angular library (ng-packagr secondary entries)
    <name>/
      index.ts         ← barrel exports + optional NgModule
      ng-package.json  ← secondary entry point {"lib":{"entryFile":"index.ts"}}
      src/
        <name>-root.directive.ts
        <name>-<role>.directive.ts   (or .ts without "directive" suffix for newer style)
      __tests__/
        <name>-root.directive.spec.ts
      stories/
        <name>.stories.ts
        <name>.ts                    ← standalone story components
apps/
  radix-storybook/     ← Storybook docs/examples; also the public site on radix-ng.com
  radix-playground/    ← Angular playground app
  radix-ssr-testing/   ← SSR smoke-test app
  radix-perf-testing/  ← performance benchmark app
  visual-regression/   ← Playwright visual and behavior specs
skills/                ← LLM consumer Agent Skills (generated from Storybook docs)
tools/
  scripts/             ← build helpers (incl. skills bundle generator)
```

## Naming conventions

| Thing             | Pattern                                        | Example                      |
| ----------------- | ---------------------------------------------- | ---------------------------- |
| Directive class   | `Rdx<Name><Role>Directive`                     | `RdxAccordionItemDirective`  |
| Newer component   | `Rdx<Name><Role>`                              | `RdxSelectRoot`              |
| Selector          | `[rdx<Name><Role>]`                            | `[rdxAccordionItem]`         |
| `exportAs`        | `rdx<Name><Role>`                              | `rdxAccordionItem`           |
| Context type      | `<Name>RootContext`                            | `AccordionRootContext`       |
| Context injectors | `inject<Name>Context` / `provide<Name>Context` | `injectAccordionRootContext` |
| NgModule          | `Rdx<Name>Module`                              | `RdxAccordionModule`         |
| Spec file         | `<name>-<role>.directive.spec.ts`              |                              |
| Stories file      | `<name>.stories.ts`                            |                              |

## Context pattern (dependency injection)

Every primitive family uses `createContext` from `@radix-ng/primitives/core`:

```ts
import { createContext } from '@radix-ng/primitives/core';

export type RdxFooRootContext = { ... };

// Second arg: docs path ('components/<name>' or 'utils/<name>') — appended to the
// missing-context error as a link to https://radix-ng.com/<path>.md. Always pass it.
export const [injectFooRootContext, provideFooRootContext] =
    createContext<RdxFooRootContext>('FooRootContext', 'components/foo');

const rootContext = (): RdxFooRootContext => {
    const instance = inject(RdxFooRootDirective);
    return { ... };
};

@Directive({
    providers: [provideFooRootContext(rootContext)],
})
export class RdxFooRootDirective { ... }
```

Child directives inject with `injectFooRootContext()` (throws) or `injectFooRootContext(true)` (returns null if absent).

## Headless state — data attributes

Expose state via `host` bindings, never inline styles (except CSS custom properties for animation dimensions):

```ts
host: {
    '[attr.data-state]': 'open() ? "open" : "closed"',
    '[attr.data-disabled]': 'disabled() ? "" : undefined',
    '[attr.data-orientation]': 'orientation()',
}
```

Use `undefined` (not `null` or `false`) to remove an attribute.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [radix-ng/primitives](https://github.com/radix-ng/primitives) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
