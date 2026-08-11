---
trigger: always_on
description: This file is read by Cursor, Codex, and other agent harnesses alongside project context. It defines how AI assistants should work in this repository.
---

# AI agents — AI DIAL UI Kit

This file is read by Cursor, Codex, and other agent harnesses alongside project context. It defines how AI assistants should work in this repository.

## Product

- **What**: `@epam/ai-dial-ui-kit` — React 19 + TypeScript component library for AI DIAL interfaces.
- **Build**: Vite library mode, Tailwind + SCSS, public API via `src/index.ts`.
- **Docs & dev**: Storybook (`npm run storybook`), tests with Vitest + Testing Library (`npm run test`).

## Agent principles

1. **Read before edit** — Open related files (component, stories, spec, types, `src/index.ts`) before changing behavior or API.
2. **Minimal diffs** — Solve the task only; no unrelated refactors or reformatting of untouched code.
3. **Verify** — After substantive changes: `npm run typecheck` (full TypeScript check; separate from ESLint), `npm run lint`, `npm run test` (and Storybook if UI/stories changed).
4. **Security** — Do not commit secrets; do not paste real tokens into chat; treat `.env` and keys as sensitive.
5. **Delegation mindset** — For large features, split: plan → implement component → stories → tests → export; ask the user if scope is unclear.

## Component work checklist

- **Naming**: Exported components use the `Dial*` prefix (e.g. `DialButton`, `DialInput`).
- **Files**: Prefer colocating `Component.tsx`, `Component.stories.tsx`, `Component.spec.tsx` under `src/components/<Name>/`.
- **API**: Extend native HTML/React props where appropriate; export prop types when consumers need them.
- **Styles**: Tailwind + existing tokens/utilities; keep a11y (labels, roles, keyboard) in mind.
- **Barrel**: Add public exports to `src/index.ts` (and `export type` for types).

## When to add or update

| Change                      | Also do                                                                         |
| --------------------------- | ------------------------------------------------------------------------------- |
| New public component        | Storybook story, spec, entry in `src/index.ts`                                  |
| Visual / interaction change | Update stories; adjust or add tests                                             |
| New interactive control     | Accessible name + target size (see [Accessibility rules](#accessibility-rules)) |
| Peer dependency surface     | Document in README or story descriptions if needed                              |
| **Breaking change**         | CHANGELOG.md entry + migration guide (see below)                                |

## Breaking changes — documentation required

A **breaking change** is any modification that requires consumers to update their code: renamed/removed props, changed component behaviour, removed exports, altered CSS token names, etc.

### 1. Find the next version

```bash
git tag --sort=-v:refname | head -1
```

This gives you the **last released** version. Breaking changes go into the **next** release, so increment the minor version (the middle number) by 1 and reset the patch to 0.

Examples: `0.10.0` → `0.11.0`, `1.3.2` → `1.4.0`.

Use this next version for all entries below.

### 2. Add a CHANGELOG entry

Open `CHANGELOG.md` and add an entry under `## [<version>]` (create the section if it does not exist). Use this format inside a `### Breaking Changes` subsection:

```markdown
### Breaking Changes

- **`ComponentName` — `oldProp` → `newProp`** — Short explanation of what changed and why.
  See [migration guide](migration-guides/<version>/<migration-name>.md).
```

Rules:

- State **what changed**, **from → to**, and **why** (alignment, consistency, accessibility, etc.).
- Link to the migration guide you will create in step 3.
- Keep it to 1–3 sentences per item.

### 3. Create a migration guide

Create a new file at `migration-guides/<version>/<migration-name>.md`.  
Use `migration-guides/_template.md` as the base and fill in all sections:

- **Why this changed** — motivation.
- **What changed** — before/after table.
- **Step-by-step migration** — concrete instructions with code examples and a grep command to locate usages.
- **Verify** — `npm run typecheck && npm run test`.

`<migration-name>` should be kebab-case and describe the change, e.g. `button-variant-prop-rename`.

### 4. Update the migration-guides index

Add a row to the table in `migration-guides/README.md`:

```markdown
| <version> | [<migration-name>](<version>/<migration-name>.md) | One-line summary |
```

## Commands reference

| Script              | Use                                                                   |
| ------------------- | --------------------------------------------------------------------- |
| `npm run typecheck` | Full `tsc` typecheck via `tsconfig.json`; **does not** replace ESLint |
| `npm run lint`      | ESLint (must pass; pre-commit hooks may run it)                       |
| `npm run test`      | Vitest with coverage                                                  |
| `npm run build`     | Library + CSS build                                                   |
| `npm run storybook` | Local docs and visual QA                                              |

## Architecture details

### Path Alias


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [epam/ai-dial-ui-kit](https://github.com/epam/ai-dial-ui-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
