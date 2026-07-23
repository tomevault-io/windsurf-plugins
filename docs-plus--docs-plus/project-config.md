---
trigger: always_on
description: Entry point for Claude Code working on **docs.plus**. Read [AGENTS.md](AGENTS.md) first — it is the durable source of truth for this repo's invariants and maintainer preferences. This file is a thin index, not a second copy.
---

# CLAUDE.md

Entry point for Claude Code working on **docs.plus**. Read [AGENTS.md](AGENTS.md) first — it is the durable source of truth for this repo's invariants and maintainer preferences. This file is a thin index, not a second copy.

## Read order

1. **[AGENTS.md](AGENTS.md)** — durable rules: package manager, git/commit policy, code quality, testing, skills, UI/theme, monorepo toolchain, publishing, and release flow. Treat it as memory: do not deviate without an explicit maintainer instruction.
2. **Package-local `AGENTS.md`** when working inside a package (e.g. [extensions/extension-hyperlink/AGENTS.md](extensions/extension-hyperlink/AGENTS.md)). Read in addition to the root file.
3. The relevant `.cursor/rules/*.mdc` for the file you are editing (see index below).

If guidance overlaps, project policy in `AGENTS.md` and `.cursor/docs/` wins; `.mdc` files are reference material for authoring.

## Hard invariants (do not violate)

These are the rules that bite hardest if missed. Full context in [AGENTS.md](AGENTS.md).

- **Bun only.** Never `npm`, `yarn`, `pnpm`, `npx` — including the install commands in published extension READMEs/CHANGELOGs (plain `bun add <pkg>`, never `npm install` or `@next` soak lines; see [AGENTS.md](AGENTS.md) §Extension Package Contract). Lockfile is `bun.lock`.
- **No commits unless asked.** No `git add`, `git commit`, `git push`, `git stash`, or `--amend` inside plan execution. End multi-task plans at a "Review checkpoint".
- **Stay in the current worktree.** Do not switch execution to another path or parallel checkout.
- **Tests are opt-in, not default.** Do not add tests unless asked, pinning a real regression, or covering dense branching logic. Prefer Cypress E2E over unit. Never write the banned shapes listed in [AGENTS.md](AGENTS.md) §Testing And Verification.
- **Prose routes through the `tech-writer` skill.** README, CHANGELOG, reports, post-mortems, PR descriptions, JSDoc.
- **JSDoc/comments ≤ 4 lines, why-not-what.** No section banners. No restating signatures.

## Cursor rules — `.cursor/rules/`

Reference material that auto-attaches in Cursor. In Claude Code, open the relevant file when the matching surface is touched.

- [design-system.mdc](.cursor/rules/design-system.mdc) — design-system pointer + cardinal rules for all webapp UI work; source of truth is [.cursor/docs/design-system.md](.cursor/docs/design-system.md).
- [react-floating-ui.mdc](.cursor/rules/react-floating-ui.mdc) — React 19.2 + `@floating-ui/react` 0.27 conventions and pitfalls.
- [supabase.mdc](.cursor/rules/supabase.mdc) — SQL authoring, Supabase migrations, generated files, RLS. Triggers on `**/*.sql`, `packages/supabase/**`, `apps/webapp/src/types/supabase.ts`.
- [tiptap.mdc](.cursor/rules/tiptap.mdc) — Tiptap/ProseMirror reference workflow for editor code under `apps/webapp/src/components/TipTap/**`, `chatroom/**`, `extension-*/**`, `hocuspocus.server/src/**`.
- [scripts-naming.mdc](.cursor/rules/scripts-naming.mdc) — script and Make-target naming. Triggers on `package.json`, `Makefile`, `.github/workflows/**`, `scripts/**`.

## Long-form policy — `.cursor/docs/`

- [design-system.md](.cursor/docs/design-system.md) — source of truth for the webapp visual language (daisyUI/Tailwind tokens, themes, elevation species, state recipes, component catalog) that `design-system.mdc` and the `design-system` skill point at.
- [scripts-naming-convention.md](.cursor/docs/scripts-naming-convention.md) — timeless source of truth that `scripts-naming.mdc` points at.

## Skills — project-local

### `.cursor/skills/` — docs.plus-specific

Open the `SKILL.md` when its trigger matches.

- [code-janitor](.cursor/skills/code-janitor/SKILL.md) — autonomous cleanup pipeline (Simplification → Abstraction → Readability → Documentation → Production-Readiness). Use for "clean up", "polish", "tidy", "production ready".
- [design-system](.cursor/skills/design-system/SKILL.md) — working protocol for any UI/theme/styling change (tokens, species, state recipes, lockstep surfaces, browser verification). Use for "theme", "colors", "dark mode", "styling", new surfaces.
- [commit-review](.cursor/skills/commit-review/SKILL.md) — group changes and draft Conventional Commit messages. Use for "review changes", "write commits".
- [tech-writer](.cursor/skills/tech-writer/SKILL.md) — senior-tech-writer voice for **all** prose: README, CHANGELOG, reports, post-mortems, PR descriptions, JSDoc prose. Other skills delegate prose work here.
- [tiptap](.cursor/skills/tiptap/SKILL.md) — Tiptap integration helper (extensions, collaboration, comments, AI, document conversion).

### `.agents/skills/` — [mattpocock/skills](https://github.com/mattpocock/skills)

Installed via `bunx skills@latest add mattpocock/skills`; lockfile: `skills-lock.json`. Run [setup-matt-pocock-skills](.agents/skills/setup-matt-pocock-skills/SKILL.md) once to wire issue tracker, triage labels, and domain docs.

Engineering highlights:

- [improve-codebase-architecture](.agents/skills/improve-codebase-architecture/SKILL.md) — deepen shallow modules, propose refactor RFCs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [docs-plus/docs.plus](https://github.com/docs-plus/docs.plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
