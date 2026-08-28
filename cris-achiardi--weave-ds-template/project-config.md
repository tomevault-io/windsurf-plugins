---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## First rule: modular context, loaded on demand

Pull the right context at the right time — not everything up front. Every piece of this system's
documentation is a **module**: each ADR, each README beside the thing it governs, each
package-level `CLAUDE.md`, each skill. Load a module only when the task actually needs the
decision or detail it holds.

In particular, **do not read all the ADRs at startup** — read the one that governs the surface you
are about to touch, when you are about to touch it.

This is deliberate **progressive disclosure**: it keeps working context lean and lets the system
scale, because the entry map stays small and you descend into a module only when you are in its
territory. This file and every doc it points to is written to be reached that way.

## What this is

A **design-system starter template**. It ships the machinery — token pipeline, component contract
system, prop glossary, Figma wiring, ADR governance, agent skills, CI — and **no components**.

That emptiness is the design, not an unfinished state. Components are built against an accepted
decision; the arc that produces one is:

> **explore → report → decide (ADR) → build**

with a skill for each of the first three steps and a gate behind the fourth.

```
packages/tokens/   @ds/tokens — DTCG JSON -> CSS custom properties + TS constants
packages/react/    @ds/react  — the component library (empty)
apps/sandbox/      fast Vite harness, in the workspace
apps/storybook/    complete on disk, deliberately OUT of the install graph
docs/research/     pre-decision: what is measurably true
docs/ADR/          post-decision: what we decided and why
.ai/maps/          the prop glossary — generated, descriptive, gated
.figma/            which design file we read, and what has been reconciled
```

**[`packages/react/CLAUDE.md`](./packages/react/CLAUDE.md) is the authority for library
internals** — component anatomy, the contract system, extraction, the gates. Read it before
authoring or modifying a component. This file covers the monorepo, the token pipeline and
governance.

## Brand it before anything else

The repo ships generic (`@ds/*`, `--ds-*`, `data-ds-*`). Run **once**, before writing components:

```bash
pnpm init-ds weave --dry   # inspect
pnpm init-ds weave         # apply, then pnpm install
```

The scope, the token prefix and the data-attribute prefix are one decision in three syntaxes and
must move together — renaming one by hand leaves a repo that builds green and is wrong. `/ds.config.json`
is the single source of truth; never hard-code a prefix anywhere else.

## Commands

```bash
pnpm dev                 # sandbox at :4300
pnpm build               # tokens, then the library
pnpm verify              # the full local gate — run this before pushing

pnpm contract <Name>     # what IS this component (source + contract, merged, no build)
pnpm contract --coverage # who is contracted
pnpm prop-map            # regenerate the prop glossary
pnpm report:paints       # token policy vs stylesheet — a REPORT, never a gate
```

`pnpm verify` chains: `format:check → typecheck → verify:contract → prop-map:check →
verify:figma → build → test`. All of it is green on a fresh clone with zero components — that is
the template's acceptance test.

## Governance lives in the ADRs — consult the one your task touches

[`docs/ADR/`](./docs/ADR/) is the governance layer. Per the first rule, do not read them all up
front; when a task touches a structural decision, open the governing record then.
[`docs/ADR/README.md`](./docs/ADR/README.md) is the index that routes you there.

Two rules make the records durable rather than a maintenance burden:

- **Decision vs contract.** An ADR records the _decision_ and stays agnostic; the schemas, scripts
  and generated files that realize it are its **contracts**, they live with the code they govern,
  and the ADR _links_ them in a `## Contract` table. This is what lets an accepted decision stay
  stable while its implementation moves.
- **Pre-v0 status policy.** Before the first release, records are edited in place and most stay
  **Draft**. Only decisions **already mechanized in code** are Accepted. At v0 the
  supersede-don't-edit discipline switches on.

Working against an accepted ADR without updating it is a defect, not a shortcut.

## The other contracts, and when to read them

| Read it when                                    | File                                                                                                                     |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Explaining any of this to a designer            | `docs/documentation/` — plain language + diagrams. Explanation, not spec: where it disagrees with a spec, the spec wins. |
| Authoring or changing a component               | `packages/react/src/components/README.md` — **the** authoring contract                                                   |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cris-achiardi/weave-ds-template](https://github.com/cris-achiardi/weave-ds-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
