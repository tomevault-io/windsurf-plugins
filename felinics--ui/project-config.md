---
trigger: always_on
description: Single source of truth for the **cross-cutting** visual and interaction decisions
---

# @felinic/ui — Design Language Contract

Single source of truth for the **cross-cutting** visual and interaction decisions
in this component library. It exists because one class of bug kept recurring:
**inventing chrome** (stray shadows / borders) and **hand-writing raw values**
instead of pulling from tokens. LLMs also tend to assume Tailwind v3 / older Vite
and reintroduce version bugs (see Motion below).

If you build or refactor a component in `packages/ui`, follow this file. When you
make a **new** cross-cutting decision, write it back here — this is a living doc.

## Web host guidance

This repository also owns the Web composition guidance consumed by host
applications. When working on a host Web page or component that uses this
library, read these files from the host repository root, in order:

1. `packages/ui/skills/web/SKILL.md` — page-level design language and workflow.
2. `packages/ui/skills/ui-owners/SKILL.md` — recurring layout-owner vocabulary.

Resolve references inside either skill relative to its own directory. Hosts
should point agents to `packages/ui/AGENTS.md`; they do not need to copy,
forward, symlink, or separately register these skills under `.agents/skills`.

## Enforcement is three layers

| Layer | Where | Role |
|---|---|---|
| **Tokens** | `src/style.css` (`@theme inline` / `:root` / `.dark`) | the ONLY place raw values live |
| **This contract** | `packages/ui/AGENTS.md` | the rules + rationale |
| **Guard** | `scripts/check-ui-contract.mjs` (wired into `mise run lint`) | mechanical block on drift |

## Compose, don't style — the one principle (locality)

LLMs are weak at CSS / visual / spatial reasoning and strong at composing by API. The whole
point of this library is to keep that weakness out of the loop: **style has exactly one home
(locality), and the layer above only composes.** An agent (or a person) expresses intent
through a component's API — `variant="destructive"` — never through raw CSS
(`bg-red-500 hover:bg-red-600`). The component is the translation layer between "what I want"
and "what CSS makes it so."

Corollary: **hand-writing CSS / injecting a class is NOT a normal move here — it is an escape
hatch.** Reaching for it means a contract is missing; the right reaction is to *add the
contract* (a `variant`, a slot, a token, a shared component), not to press the hatch and paint
over the component.

**Ownership — every concern has exactly ONE home:**

| Concern | Its only home | Who must never touch it |
|---|---|---|
| Raw values (color / shadow / radius / size) | tokens (`style.css` `:root` / `.dark` / `@theme`) | any `.vue`, any app page |
| Interaction chrome (hover / press / focus / open) | `style.css @layer components`, keyed off `data-*` | any `.vue`, any app page |
| Layout + resting state (height / padding / radius / rest color) | the component `.vue` + `cva` | app pages don't re-write it |
| Composition (arranging components into a screen) | app pages + shared composition components | — |
| Business / orchestration | page logic | components embed no business assumption |

The test: for any value, *"where does it live?"* must have exactly **one** answer. The same
value in two homes is debt. *How* these homes fight the moment you ignore the boundary is the
next section.

**Why locality is the whole game — one place to change, everyone benefits.** Tokens and
components exist so a maintainer edits ONE value and every caller updates for free. Every
magic string a caller must copy to use a component correctly — a layout-class recipe, an
unenforced prop pairing — breaks that guarantee: the value is now pinned at N call sites,
out of the maintainer's reach, and each copy can mis-transcribe a fragment and resurrect a
solved bug. So the acceptance test for a NEW component is: *a caller who never read the
implementation fills in content only — zero layout/appearance CSS.* Expose knobs as
**enumerated props** (`width="2xl" | "3xl"`), never free-text classes — the enum forces the
next rung to be added HERE, deliberately, not invented per page. And when you meet an
existing component that fails this test, tell the human explicitly instead of copying the
recipe one more time — that report is the highest-value defect signal this contract has.
(Case study: `DialogPanel` — the focused-dialog shell was a hand-copied
`max-h-[80dvh] grid-rows-[…]` string until it became a component.)

## The cascade has four override planes (read before you "just add a class")

The recurring frustration — *"my CSS does nothing / something else wins / the conflict is
invisible"* — is not because the CSS is messy. It is because a final style here has to hold
on **four independent override planes at once**. Touch one without knowing the others and the
result silently changes. This is the mental model that the rest of this file assumes; an agent
that lacks it keeps re-introducing the same class of bug.

**The stack, bottom → top:**

```
5  app pages (apps/web)         compose components; pass ONLY tokens + variant/size
4  .vue + cva (index.ts)        ONLY layout + resting color (height/padding/radius/rest text)
3  style.css @layer components   ALL interaction chrome (hover/press/focus/open), keyed off data-*

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [felinics/ui](https://github.com/felinics/ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
