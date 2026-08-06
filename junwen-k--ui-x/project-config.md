---
trigger: always_on
description: Guidance for AI agents (and humans) working in **junwen-k/ui-x**. These are
---

# AGENTS.md

Guidance for AI agents (and humans) working in **junwen-k/ui-x**. These are
principles to apply with judgment, not boxes to tick — when a case doesn't fit,
reason from the "why" here and say so.

---

## What this project is

ui-x is a **shadcn-style component registry** — a natural extension of
[shadcn/ui](https://ui.shadcn.com), not a fork. We ship the components shadcn
doesn't (yet) have, built and styled so they feel like they came from shadcn
itself: a user should be able to `npx shadcn add` one of ours next to theirs and
not feel a seam.

Everything below follows from that stance.

---

## Principles

- **Compose shadcn, don't reinvent it.** Depend on shadcn's components; never
  copy or restyle them. When shadcn ships a first-class version of something we
  filled in, that component is **superseded** and removed in favor of the
  official one (see `apps/v4/content/docs/changelog.mdx`).
- **Lean on great headless libraries; author behavior only when none exists.**
  Reach for a popular, well-maintained headless library before writing
  interaction logic yourself — `react-dropzone`, `react-phone-number-input`,
  `timescape`, `react-day-picker`, `@dnd-kit`, `frimousse`, `virtua`. We hand-
  author behavior (on [Base UI](https://base-ui.com)) only where no good library
  fits — e.g. the password visibility toggle.
- **Match shadcn's conventions.** `data-slot` attributes, CVA variants,
  `render`-prop composition, file layout, naming, prose tone. Unsure how to do
  something? Find the closest shadcn/ui component and mirror it.
- **Model the API deliberately** — see [the wrapping principle](#the-wrapping-principle),
  the one rule most worth getting right.

## Hard rules

- **Base UI, never Radix.** Base UI is the headless foundation shadcn moved to;
  it's what we build on when we author unstyled behavior. Never add `@radix-ui/*`.
- **One style: `base-nova`** (set in `apps/v4/components.json`). Pull shadcn core
  in via the CLI — `npx shadcn@latest add <name>` — never hand-transcribe
  classes. Base-nova payloads sometimes run ahead of the released Tailwind CSS
  (new `cn-*` utilities); if you must vendor a class, confirm it exists in the
  released `tailwind.css` first.
- **Conventional Commits**, enforced by commitlint.
- **Never self-merge a PR**, force-push, or run destructive git without explicit
  confirmation. Branch, push, open the PR, leave it for the maintainer.
- **Verify state directly** — read the file, check the branch — rather than
  assuming what's frozen or already done.

---

## How components are built

There's no single mold. A component takes the simplest shape that fits:

- **Styled composition** — styling over shadcn core and/or a headless library,
  composed directly, no separate primitive (`emoji-picker`, `sortable`,
  `wheel-picker`, `confirmer`).
- **Primitive + styled** — an unstyled `<name>-primitive.tsx` layer plus a
  styled `<name>.tsx` that composes it with shadcn core. Author a primitive when
  the headless behavior is worth publishing on its own — usually a thin
  Base UI-flavored adapter over a library (`dropzone` → react-dropzone,
  `phone-input` → react-phone-number-input, `date-time-field` → timescape), or
  hand-written on Base UI when no library fits (`password-input`).

`apps/v4/src/registry/new-york/ui/date-picker.tsx` is the reference to study
before designing a new component.

**Import boundaries:**

- shadcn core → `@/components/ui/*` (added via CLI; **not** part of our registry).
- ui-x components & primitives → `@/registry/new-york/{ui,components}/*`.
- Demos import the styled component from `@/registry/new-york/ui/<name>`.

(The folder is named `new-york/` for historical reasons — the _style_ is `base-nova`.)

---

## Anatomy & API consistency

A new component's API should feel like one a consumer already knows. Before
naming anything, look at the closest shadcn primitive and the nearest ui-x
sibling, and match them.

- **Parts are `<Component><Part>`**, PascalCase, exported root-first in
  composition order — the same tree the docs "Usage" block shows
  (`PhoneInput` → `PhoneInputCountrySelect` → `PhoneInputCountrySelectContent`).
- **`data-slot` is the kebab-case of the exported name** — nothing invented
  (`PhoneInputCountrySelectValue` → `data-slot="phone-input-country-select-value"`).
- **Reuse the established part vocabulary.** A surface is `*Content`, an opener
  `*Trigger`, an option `*Item`, the selected display `*Value`, the field
  `*Input`. Don't coin a new noun for a role that already has one — a consumer
  should be able to guess a part's name from what it does.
- **Mirror the underlying anatomy so composition transfers.** When a part wraps
  a shadcn primitive, keep the same shape and let it compose with that
  primitive's own parts — `PhoneInputCountrySelect` sits inside Select's own
  `SelectTrigger`, so anyone who knows Select already knows this.

When a new component and an existing one solve the same sub-problem, they should
read the same way. Consistency across the set beats a locally clever API.

---

## The wrapping principle

When you expose a sub-component, choose deliberately between **wrapping and
re-exporting** a part vs. letting the consumer **compose it from the outside**.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [junwen-k/ui-x](https://github.com/junwen-k/ui-x) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
