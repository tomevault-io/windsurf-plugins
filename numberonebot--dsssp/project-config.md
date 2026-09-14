---
trigger: always_on
description: - **Language** — all code comments, documentation files, commit messages, and PR descriptions must be written in **English**. No exceptions. (Chat with the author may be in any language — this rule is about committed artifacts.)
---

# DSSSP Library — AI Instructions

## Code Style

### General

- **Language** — all code comments, documentation files, commit messages, and PR descriptions must be written in **English**. No exceptions. (Chat with the author may be in any language — this rule is about committed artifacts.)
- **Named exports only** — no default exports in library code. _Exception:_ Storybook stories and `.mdx` docs, where the CSF format requires `export default meta`.
- **camelCase everywhere** — including constants; no SCREAMING_CASE.
- **Imports** — no automatic import-sorter is configured (ESLint here does not sort imports). Don't reorder imports wholesale; follow the grouping already present in the file you're editing (Node built-ins → external packages → internal relative modules).
- **ESLint clean** — no ESLint errors or warnings in committed code — fix immediately, never suppress. The Husky pre-commit hook runs `npm run lint` (`tsc && eslint .`) and **blocks the commit on any error** (e.g. unused variables error via `@typescript-eslint/no-unused-vars`).
- **Dead code** — unused imports, variables, props, and files must be deleted immediately as part of every task. Never ask for confirmation. Never leave anything "just in case".
- **Validation during implementation** — while working, treat **TypeScript** type errors as the signal to act on. Prettier formats on save and ESLint auto-fixes fixable issues on save (configured in `.vscode/settings.json`; the same formatting also runs via `npm run format` and the pre-commit hook), so don't hand-format or manually fix mechanical lint nits. Any remaining (non-auto-fixable) ESLint error still **blocks the commit** via the pre-commit hook — resolve those before committing.
- **State** — React Context only: the library ships `GraphProvider` and the `useGraph` hook. No Redux/Zustand.
- **Boy Scout Rule** — leave every file you touch cleaner than you found it. Fix naming, split oversized components, remove dead code — within the file being changed, not across adjacent code.
- **No confabulation** — never present an assumption as a verified fact. If you haven't checked (read a file, run a search, run the build), you don't know. Say "I haven't verified …" and check — don't guess and assert. This applies especially to: the exported API surface, prop semantics, and any rendering/runtime behaviour you haven't confirmed in the source.

### Naming

Follow **Intention-Revealing Names** (Clean Code, Robert Martin): names should answer _"what is this for?"_, not _"what type/state is this?"_.

**No `is`/`has`/`can`/`should` prefixes — hard rule.** These are Systems Hungarian notation: they encode the value's type (boolean) into the name instead of its meaning. TypeScript already carries the type. Applies to:

- boolean variables — `loading`, not `isLoading`; `dragging`, not `isDragging`; `open`, not `isOpen`
- boolean props — `<Modal open />`, not `<Modal isOpen />`
- boolean fields on types we own — e.g. `curve.dotted`, not `curve.isDotted`; `filter.bypassed`, not `filter.isBypassed`
- predicate / type-guard functions — `pointInBounds`, `frequencyInRange`, `matchesFilterType`, not `isPointInBounds`, `isFrequencyInRange`, `doesMatchFilterType`

**Exceptions (closed list):**

- fields on external API responses or third-party types we cannot rename
- standard library / ecosystem names: `Array.isArray`, `Number.isFinite`, `Number.isNaN` (the library itself uses `isNaN` in `math.ts`)

**Write-time check:** _"Am I about to type `is`, `has`, `can`, or `should` as a prefix? → rename before typing the rest."_ If the non-prefixed name feels unclear, fix the noun (`loading` vs `flag`), don't add back the prefix.

**Keep names compact.** Drop modifiers that add no information given the context. If there is only one graph in scope, it is just `graph` — not `activeGraph` or `currentGraph`. If a boolean describes the only relevant state, name it for the state (`dragging`) not the mechanics (`isCurrentlyDragging`). Shorter names that still answer _"what is this for?"_ are always preferred.

**Keep names uniform with the public API.** Match the field names of the types you work with instead of inventing synonyms. `GraphFilter` uses `freq`, `gain`, `q`, `type` — pass a filter's frequency as `freq`, not `frequency` or `qFactor`. (Where a type genuinely uses the full word — e.g. `Magnitude.frequency` — follow that type.)

**Match parameter names to the destination field.** When a parameter is always passed into a specific object property, name it to match that property so shorthand works — e.g. React `key`/`ref`, or a `GraphFilter` field: name the param `freq` (then `{ freq }`), not `frequency`, which forces `{ freq: frequency }`.

**Destructure before repeated access.** When `obj.field` appears 2+ times in a function body, destructure once at the top — `const { freq, gain, q, type } = filter` — instead of repeating `filter.freq` at each call site.

### Component Structure

Start every new component as a **single file**. Keep it there as long as it satisfies single responsibility with no nested render helpers.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NumberOneBot/dsssp](https://github.com/NumberOneBot/dsssp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
