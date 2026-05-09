---
trigger: always_on
description: This file is read by Claude Code child agents and defines the project-level contract for automated work on this repository.
---

# ARS — Agent Working Agreement

This file is read by Claude Code child agents and defines the project-level contract for automated work on this repository.

---

## Project Context

**ARS (Agentic Remotion Studio)** is a Claude Code plugin + starter repo for building Remotion-based video episodes. It is structured as a registry-first card system where every content type is registered via `CardSpec` and rendered through a single dispatch path.

Key paths:
- `src/engine/` — Remotion rendering engine (cards, layouts, scenes, shared primitives)
- `src/engine/cards/registry.ts` — Runtime card registry (Vite/browser only, `import.meta.glob`)
- `src/engine/shared/card-catalog.ts` — CLI-only static card metadata (`CARD_CATALOG`, not a runtime registry)
- `src/engine/shared/types.ts` — Source of truth for `Episode`, `Step`, `ShellConfig`, `SeriesConfig`
- `src/types/studio-intent.ts` — Source of truth for Studio intent schema shared by Studio and CLI
- `src/studio/studio-intents.ts` — Node-side helpers for creating, listing, and resolving Studio intents
- `src/episodes/template/` — Template series used for smoke testing
- `cli/` — Node.js CLI (`npx ars <command>`)
- `plugin/skills/` — Claude Code slash commands (shared via the ARS plugin)

---

## Operating Principles

- **Read before editing.** Never modify a file you haven't read in the current session.
- **Prefer deletion over addition.** The right fix is usually removing the wrong thing, not adding a workaround.
- **Reuse existing patterns first.** Before introducing a new abstraction, verify no existing utility covers it.
- **Write a cleanup plan before refactoring.** For changes touching 3+ files, draft a plan first.
- **Keep diffs small and reversible.** One logical change per commit.
- **No new dependencies without explicit user approval.**
- **Verify after every change.** Run `npm run lint` after edits. TypeScript errors are blocking.

---

## Agent PR Contract

Most ARS changes are authored by coding agents. Treat every PR as an artifact that another agent and a human maintainer can audit later.

Before opening or preparing a PR:
- Confirm the exact repo is `/Users/dylan_lu/cowork-workspace/ars` or the intended ARS source checkout. Do not include dirty changes from consumer/dogfood repos such as generated episode output unless the task explicitly says so.
- Keep one logical change per PR. If the work mixes docs, runtime behavior, generated repo support, and dogfood episode output, split it or explain why it cannot be split.
- Read `CONTRIBUTING.md` and include its PR expectations in the PR body.
- For any release-sensitive surface, include a migration decision: `No migration required` or concrete `ars update` work.
- For Studio/UI changes, include screenshots, browser notes, or a concise manual verification note.
- For package-surface changes, run and summarize `npm pack --dry-run`.
- Never publish to npm, create public tags, or push release tags without explicit maintainer confirmation.

Recommended agent-authored PR body:

```markdown
## Summary
- ...

## Compatibility / Migration
- Release-sensitive surfaces touched:
- Migration decision:

## Verification
- [ ] npm run lint
- [ ] npm run build:studio
- [ ] npm test
- [ ] npx ars episode validate template/ep-demo
- [ ] npm pack --dry-run (when package/release surface changed)

## Notes
- Screenshots / Studio notes:
- Known risks:
```

---

## Card System Contract

Every card type MUST follow this contract. Violating it breaks the registry-first renderer path.

### Required structure for a core engine card
```
src/engine/cards/<type>/
  component.tsx   # React component implementing CardRenderProps<TData>
  spec.ts         # exports `cardSpec satisfies CardSpec<TData>`
  schema.ts       # (optional) Zod schema for data validation
```

### Required structure for a series-scoped custom card
```
src/episodes/<series>/cards/<type>/
  component.tsx
  spec.ts         # exports `cardSpec satisfies CardSpec<TData>`
```

### spec.ts contract
```typescript
export const cardSpec = {
  type: "my-type",          // unique, kebab-case
  title: "Human Label",
  description: "One-line description of what this card renders.",
  component: MyCardComponent,
  schema: MyZodSchema,      // optional but strongly recommended
} satisfies CardSpec<MyCardData>;
```

### Invariants
- `cardSpec.type` must be unique within series cards. A series-scoped card **may** share a `type` with an engine card to fully replace it — the series card silently overrides the engine card in that repo's registry.
- Series-scoped cards still live under `src/episodes/<series>/cards/`, but `cardSpec.type` should use a globally unique bare name unless intentionally overriding an engine card (e.g. `normal-distribution` for a new card, `markdown` to replace the built-in).
- Core engine cards (`src/engine/cards/`) must never import from `src/episodes/`.
- `card-catalog.ts` (`CARD_CATALOG` array) is CLI-only. Never import it in browser/Remotion code.
- `cards/registry.ts` (`CARD_REGISTRY` Map) is Vite/browser-only. Never import it in CLI/Node code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dnplus/ars](https://github.com/dnplus/ars) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
