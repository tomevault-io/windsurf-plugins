---
trigger: always_on
description: Runtime-neutral guidance for AI coding agents in this repo. This file is the always-loaded
---

# Repository instructions

Runtime-neutral guidance for AI coding agents in this repo. This file is the always-loaded
**map**; deep per-area detail lives in `.agents/rules/*.md`. Before editing a path, read the
rules whose `paths` glob matches it.

## Personal style — defer to lailai.skill

lailai's **general, cross-project** style — Chinese voice and wording, Markdown, LaTeX math,
OI C++, design principles (统一·简约·现代), who he is and how he decides — lives in the
**lailai.skill** submodule at [`.agents/skills/lailai-skill/`](.agents/skills/lailai-skill/SKILL.md).
Read its `SKILL.md`, then the relevant `references/` / `profile/`, for any task touching voice,
writing, code, or design.

**This repo's `.agents/` holds portable project config.** It does not duplicate the general
rules; where `.agents/rules/*.md` covers only the site-specific slice, it points to the skill
for the rest. Runtime-specific directories are compatibility adapters only.

Init the submodule after cloning: `git submodule update --init`. Update it later with `git submodule update --remote .agents/skills/lailai-skill`.

## Project

Source for [lailai's personal website](https://lailai.one) — Docusaurus 3 (TypeScript), Node `>=20` (CI uses Node 24). Deployed via GitHub Actions to GitHub Pages and rsynced to a custom server (`.github/workflows/deploy.yml`).

## Commands

```bash
npm start                 # Dev server (default English locale)
npm run start:zh-Hans     # Dev server in Simplified Chinese
npm run build             # Production build into ./build
npm run serve             # Serve the built site locally
npm run clear             # Clear Docusaurus cache (.docusaurus)

npm run i18n              # Regenerate translation JSON for zh-Hans
npm run format            # Prettier write across the repo
npm run lint              # ESLint over src/ (flat config, eslint.config.mjs)
npm run typecheck         # tsc, no emit
npm run check             # i18n + format + lint + typecheck — run before every commit
```

There is no test runner. `npm run check` is the gate.

**Lint layer is deliberately lean** (`eslint.config.mjs`, flat config, `src/` only): Prettier owns formatting and tsc owns types, so ESLint carries no stylistic rules — just what the other two can't see. That's `typescript-eslint` recommended (minus `no-explicit-any` and `no-require-imports`, both of which fight legitimate Docusaurus/webpack patterns here) plus the two **classic** React Hooks rules (`rules-of-hooks` error, `exhaustive-deps` warn). Do **not** switch react-hooks to its v7 `recommended-latest` — the React-Compiler rules it adds (`set-state-in-effect`, `refs`, `immutability`) flag valid patterns and drown the signal. Unused-var escape hatch: `_`-prefix.

## Rules index

Path-scoped detail — before editing, read each file whose scope matches the target paths.
Don't restate their content here; extend the file itself.

| Rule                                                               | Scope                                | Covers                                                                                                          |
| ------------------------------------------------------------------ | ------------------------------------ | --------------------------------------------------------------------------------------------------------------- |
| [`.agents/rules/components.md`](.agents/rules/components.md)       | `src/**` ts·tsx·css                  | `laikit` inventory, CSS-Module layout & rule ordering, hover-motion limits, text-overflow, MDX widgets          |
| [`.agents/rules/i18n.md`](.agents/rules/i18n.md)                   | `src/**`, `i18n/**`                  | `translate()` workflow, five-prefix taxonomy, key shapes, orphan cleanup                                        |
| [`.agents/rules/comments.md`](.agents/rules/comments.md)           | `src/**`, `*.ts`                     | code-comment style (site-specific slice)                                                                        |
| [`.agents/rules/datetime.md`](.agents/rules/datetime.md)           | `src/**`, `docusaurus.config.ts`     | instant/calendar/duration semantics, storage offsets, visitor display zones, API boundaries                     |
| [`.agents/rules/writing-style.md`](.agents/rules/writing-style.md) | `blog/**`, `docs/**`, translated MDX | frontmatter, headings, tone, MDX widgets, math, images, links, solution template                                |
| [`.agents/rules/solution-sync.md`](.agents/rules/solution-sync.md) | `blog/solution/**`                   | 题解 → 洛谷: thin pointer to skill's `luogu-solution.md` (full flow + red lines) + project mirror/summary rules |

## Architecture

### Directory map

| Path                                    | Purpose                                                                                                                                                                                                                                |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lailai0916/lailai0916.github.io](https://github.com/lailai0916/lailai0916.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
