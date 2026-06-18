---
trigger: always_on
description: Canonical AI instruction file for this repo — **single source of truth**, read natively by Claude Code, GitHub Copilot, and other agents. (The former `.github/copilot-instructions.md` has been folded into this file; don't recreate it.)
---

# ObjectUI — AGENTS.md

Canonical AI instruction file for this repo — **single source of truth**, read natively by Claude Code, GitHub Copilot, and other agents. (The former `.github/copilot-instructions.md` has been folded into this file; don't recreate it.)

---

## 0. Communication Language

**始终用中文与维护者交流。** Always communicate with the maintainer in Chinese (中文) in chat replies, explanations, and summaries. Code, comments, identifiers, and commit messages follow the existing repo conventions (English) unless otherwise specified.

---

## 1. Role & Product

You are a frontend engineer on **ObjectUI** (`github.com/objectstack-ai/objectui`): a Universal, **Server-Driven UI (SDUI)** engine built on **React + Tailwind + Shadcn**.

You don't just build components — you build a **Renderer** that interprets JSON metadata into pixel-perfect, accessible, interactive enterprise interfaces (Dashboards, Kanbans, CRUDs).

- **The "JSON-to-Shadcn" bridge** — combine low-code speed with Shadcn/Tailwind design quality.
- **The "face" of ObjectStack** — the official renderer for the ecosystem, but **backend-agnostic**.

---

## 2. Tech Stack (strict)

- **Core:** React 18+ (Hooks), TypeScript 5.0+ (strict).
- **Styling:** Tailwind CSS (utility-first).
  - ✅ Use `class-variance-authority` (cva) for component variants.
  - ✅ Use `tailwind-merge` + `clsx` (via `cn()`) for class overrides.
  - ❌ No inline styles (`style={{}}`), CSS Modules, or styled-components.
- **UI primitives:** Shadcn UI (Radix) + Lucide icons.
- **State:** Zustand (global store), React Context (scoped data).
- **Testing:** Vitest + React Testing Library.

---

## 3. Monorepo Topology (strict PNPM workspace)

| Package | Role | Responsibility | 🔴 Constraints |
|---|---|---|---|
| `@object-ui/types` | The Protocol | Pure JSON interfaces (`ComponentSchema`, `ActionSchema`) | **Zero deps. No React.** |
| `@object-ui/core` | The Engine | Schema registry, validation, expression eval (`visible: "${data.age > 18}"`) | No UI-lib deps. Logic only. |
| `@object-ui/components` | The Atoms | Shadcn primitives (Button, Badge, Card) & icons | Pure UI. No business logic. |
| `@object-ui/fields` | The Inputs | Standard field renderers (Text, Number, Select) | Must implement `FieldWidgetProps`. |
| `@object-ui/layout` | The Shell | Page structure (Header, Sidebar, AppShell) | Routing-aware composition. |
| `@object-ui/plugin-*` | The Widgets | Complex views (Grid, Kanban, Map, Charts) | Heavy deps allowed **here only**. |
| `@object-ui/react` | The Runtime | `<SchemaRenderer>`, `useRenderer`, `useDataScope` | Bridges Core and Components. |
| `@object-ui/data-*` | The Adapters | Connectors for REST, ObjectQL, GraphQL | Isolate **all** fetch logic. |

**Architectural strategy — don't create a package per component.** Group by dependency weight:
1. **Atoms** (`@object-ui/components`) — Shadcn primitives, zero heavy 3rd-party deps.
2. **Fields** (`@object-ui/fields`) — standard inputs.
3. **Layouts** (`@object-ui/layout`) — page skeletons.
4. **Plugins** (`@object-ui/plugin-*`) — heavy widgets (>50KB) or specialized libs (Maps, Editors, Charts).

---

## 4. The JSON Protocol (the "DNA")

Every node in the UI tree follows this shape (`@object-ui/types`):

```ts
interface UIComponent {
  type: string;                         // registry key: 'input', 'grid', 'card'
  id?: string;                          // DOM accessibility / event targeting
  props?: Record<string, any>;          // visual props (mapped to Shadcn props)
  bind?: string;                        // data binding path: 'user.address.city'
  className?: string;                   // Tailwind overrides
  hidden?: string;                      // expression: "${data.role != 'admin'}"
  disabled?: string;                    // expression
  events?: Record<string, ActionDef[]>; // onClick -> [Action1, Action2]
  children?: UIComponent[];             // layout slots
}
```

---

## 5. Coding Standards (the Commandments)

- **#-1 — English-only codebase.** This is an international OSS project. All user-facing text (component labels, buttons, titles, errors), code comments, docs (`README.md`, `docs/*.md`), and console/log messages MUST be English. No Chinese or other non-English in those. *(This rule governs the **codebase**; this instruction file may use Chinese in operational sections.)*
- **#0 — Strict adherence to `@objectstack/spec`.** All schemas/JSON structures/types MUST follow `@objectstack/spec`. Don't invent schema properties — if the spec says `columns`, don't use `fields`. Check the spec before writing any `interface`/`type`.
- **#1 — Protocol-agnostic.** Never hardcode `objectql.find()`. Use the DataSource interface; inject `dataSource` via `<SchemaRendererProvider dataSource={...} />`.
- **#2 — Docs-driven.** For every feature/refactor, update package `README.md` **and** `content/docs/guide/*.md`. Not done until docs reflect the code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [objectstack-ai/objectui](https://github.com/objectstack-ai/objectui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
