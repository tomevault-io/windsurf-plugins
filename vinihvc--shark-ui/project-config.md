---
trigger: always_on
description: Use when the control has no visible text (icon-only buttons, unlabeled search fields where appropriate).
---

# Agent guide: Shark UI

This document is the **in-repo** source of truth for contributors and coding agents working on [Shark UI](https://shark.vini.one): a **shadcn-style component registry** built on [**Ark UI**](https://ark-ui.com) (`@ark-ui/react`), **Tailwind CSS v4**, and **Next.js**.

Use it when adding or editing primitives, registry examples, docs MDX, or when adapting snippets from Radix/shadcn ecosystems.

---

## 1. What to read first

| Need | Location |
|------|----------|
| Public API, anatomy, install | `content/docs/components/<name>.mdx`, `content/docs/utilities/<name>.mdx` |
| Working compositions | `registry/react/examples/<name>/example-*.tsx` |
| Implementation & Ark wiring | `registry/react/components/<name>.tsx` |
| Published registry JSON (CLI) | `public/r/<name>.json` (from `pnpm registry:build`) |
| Per-item build metadata | `registry/manifest/<name>.ts` |
| LLM-oriented surfaces | `app/(llms)/`, `lib/llms.ts`, `lib/llms-registry-examples.ts` |
| Deeper agent rules | `skills/shark-ui/SKILL.md` and `skills/shark-ui/references/` |
| RTL / locale | `content/docs/(root)/rtl.mdx` |

Extended detail for agents: `skills/shark-ui/references/component-registry.md`, `skills/shark-ui/references/rules/composition.md`, `skills/shark-ui/references/rules/forms.md`, `skills/shark-ui/references/rules/styling.md`, `skills/shark-ui/references/rules/migration.md`.

---

## 2. Import paths (this repository)

- **Components and examples in this repo:** `@/registry/react/components/<name>` (not deep relative imports like `../../components/...`).
- **Utilities:** `@/lib/utils` (`cn`, etc.) unless a doc specifies otherwise.
- **Consumer projects** after `npx shadcn@latest add @shark/<name>`: follow the installation section in docs (commonly `@/components/ui/...`).

---

## 3. Registry examples

Examples live under:

`registry/react/examples/<component>/example-<topic>.tsx`

**Form guides** (React Hook Form, TanStack Form, Formisch) use a second level: `registry/react/examples/form/<rhf|tanstack|formisch>/example-<topic>.tsx`.

Conventions:

- Prefer a **default export** that is a small demo component (e.g. `ButtonDemo`, `ComboboxDemo`), matching existing files in the same folder.
- Add **`"use client"`** when the example uses hooks, browser APIs, or interactive state that is not purely static markup.
- Keep each file focused on **one** scenario; split variants across `example-*.tsx` files.
- **Mirror docs and source:** exports, child structure, and prop names must match `content/docs/components/<name>.mdx` and `registry/react/components/<name>.tsx`.

CLI install pattern for consumers (from docs):

```bash
npx shadcn@latest add @shark/<component>
```

Registry build (maintainers):

```bash
pnpm registry:build
```

Prebuild runs `registry:build` automatically via `package.json` `prebuild`.

---

## 4. Ark UI composition: triggers and `asChild`

Shark follows **Ark UI** patterns. For triggers and items that should merge onto a host element (e.g. `Button`, `NextLink`), use **`asChild`** with a **single** child, as in the Menu examples:

```tsx
<MenuTrigger asChild>
  <Button variant="outline">Open</Button>
</MenuTrigger>
```

Do **not** assume other headless libraries’ APIs (e.g. `render={...}` on triggers) without verifying Shark docs and source.

Overlay surfaces use Shark’s named parts (e.g. `DialogContent`, `DialogHeader`, `SheetContent`, …) — follow each component’s MDX anatomy, not another design system’s `*Popup` / `*Panel` naming.

---

## 5. Collections: Select, Combobox, Listbox

Many list primitives expect an Ark **collection** (`createListCollection`, `useListCollection`, …), not a loose `items` prop where the docs show a collection.

**Combobox:** use `useFilter` + `useListCollection`, pass `collection` to the root, call `filter(inputValue)` from `onInputValueChange`, and render `collection.items.map(...)` inside the list. See `registry/react/examples/combobox/example-default.tsx`.

**Select:** pass `collection={...}` and map `collection.items` inside the content, inside **`SelectGroup`** when grouping (see `skills/shark-ui/references/rules/composition.md`).

---

## 6. Migrating from shadcn / Radix mental models

High-level rules:

1. **Do not** change only import paths on a shadcn snippet and assume it works.
2. Confirm **child structure**, **controlled vs uncontrolled** defaults, and **lazy mount** behavior from MDX + source.
3. Use `skills/shark-ui/references/rules/migration.md` for side-by-side patterns (Select, Toggle Group, Accordion, Input OTP, etc.).

Examples of common shifts:

- **Toggle group:** `type="single"` / `multiple` from Radix map to Shark’s value arrays and API — see migration doc.
- **Accordion:** Shark defaults differ from Radix `type="single"` / `collapsible` — see migration doc.
- **Input OTP:** no `input-otp` package; use Shark `InputOTP` / `InputOTPSlot` / `InputOTPSeparator` (no `InputOTPGroup`).

---

## 7. Icons (`lucide-react`)

- Import **specific** icons: `import { PlusIcon, XIcon } from "lucide-react"`.
- Do **not** use numeric **`size`** on icons; use Tailwind `className="size-4"` (or parent styles) when needed.
- **Decorative** icons (label or button text already explains the action): `aria-hidden="true"`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vinihvc/shark-ui](https://github.com/vinihvc/shark-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
