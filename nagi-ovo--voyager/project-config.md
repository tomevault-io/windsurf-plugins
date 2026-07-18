---
trigger: always_on
description: bun install                # Setup
---

# CLAUDE.md - Gemini Voyager

## Commands

```bash
bun install                # Setup
bun run dev:chrome         # Dev (also: dev, dev:firefox, dev:safari, dev:chrome-open)
bun run build:chrome       # Build Chrome (also: build, build:firefox, build:safari)
bun run build:edge         # Edge package (runs Chrome build, adjusts manifest, creates zip)
bun run build:all          # Build Chrome + Firefox + Safari (does not include Edge)
bun run test               # Test (also: test:watch, test:ui, test:coverage)
bun run typecheck          # Type check
bun run lint               # Lint and auto-fix
bun run format             # Format
bun run bump               # Bump package/manifest versions and run format
bun run docs:dev           # Docs dev server
bun run docs:build         # Build docs
bun run docs:preview       # Preview built docs
```

## Core Rules

Path-scoped rules live in `.claude/rules/` and load automatically by glob: `typescript.md` (src/**/*.ts(x)), `content-scripts.md` (src/pages/content/** and public/contentStyle.css), `i18n.md` (src/locales/**), `high-complexity.md` (core storage/sync services plus folder/export services and content modules).

Project-wide rules (always in effect):

1. **Never modify `dist_*` folders directly.**
2. **Never commit `.env` or secrets.**
3. **Never grant a page or feature direct `<all_urls>` permission.** If it is truly unavoidable, discuss it with the user first and get explicit approval before implementation.
4. **When adding Material Symbol icons**, the popup uses the bundled font in `public/fonts/`; verify the glyph exists locally or update the bundled font assets. Do not add a remote Google Fonts URL.
5. **For GitHub issue/PR/comment work, prefer `gh` as the source of truth** instead of browser scraping or unstable connectors.
6. **After fixing an issue with a pushed `Fixes #xxx` / `Closes #xxx` commit or PR**, leave a short GitHub comment in the reporter's language: the fix has landed, it will be available in the next version, and they are welcome to reopen the issue if the problem remains.
7. **Default push target**: when asked to push without explicit branch/PR instructions, push a fast-forward update to `origin/main`. Never force-push unless explicitly requested.
8. **Regression notes are required context.** Before implementing a non-trivial feature or bug fix, skim `.github/docs/REGRESSION_NOTES.md` for related traps. After fixing a non-obvious bug that future maintainers could plausibly repeat, proactively suggest adding a short entry with the protecting test or verification command.
9. **Do not force-refresh Gemini's SPA for feature navigation.** For Gemini content-script navigation, prefer native in-app links first and History API / router events as the fallback. Do not introduce `location.assign`, `location.href`, or `location.reload` for normal conversation/session navigation unless the user explicitly accepts a full page reload. Preserve `/u/<index>/...` account scope when constructing routes.

## Verification (run before declaring done)

1. `bun run typecheck` — after any `.ts`/`.tsx` change
2. `bun run lint` — before finishing; note this runs `eslint . --fix`, so inspect resulting changes
3. `bun run test` — all tests pass
4. `bun run build:chrome` — builds without error
5. `bun run docs:build` — after any `docs/**/*.md` or `docs/.vitepress/**` change
6. `bun run docs:dev` — after docs changes when preview is needed, start in background so user can preview in browser before committing
7. New features/fixes must include tests

## Commit Format

Conventional Commits: `<type>(<scope>): <imperative summary>`

- Types: `feat`, `fix`, `refactor`, `chore`, `docs`, `test`, `build`, `ci`, `perf`, `style`, `revert`, `deps`, `ux`
- Scope: short, feature-focused, lowercase when possible; commitlint currently enforces lowercase scopes and a 100-character header limit
- Summary: imperative, preferably lowercase, no trailing period
- If the commit relates to a GitHub issue or discussion, include `Closes #xxx` or `Fixes #xxx` in the commit body or PR description
- GPT-5.5-authored commits should include `Co-authored-by: GPT-5.5 <codex@openai.com>` in the commit body.

## Design Principles

1. **KISS.** Implement the minimum interpretation of requirements. Never combine orthogonal features (e.g., "fade" and "thin") without explicit confirmation.
2. **Backward compatibility is iron law.** Zero destructiveness to user data (especially `localStorage`).
3. **Data structures first.** Eliminate special cases by redesigning data, not adding branches.
4. **For visual/CSS changes:** describe expected rendering, verify alignment/centering/spacing in both light and dark themes, and check external resources (icon fonts, CDN links).
5. **For ambiguous requirements:** implement the minimal version first. Ask before adding scope.
6. **Grep for a sibling precedent before adding a new primitive.** Body-level popover, global listener, CSS overlay — there is almost always an existing `gv-pm-*` analogue (e.g., `.gv-pm-confirm` for body-appended popovers) already wired into close-outside handlers, teardown, and theme overrides. Copy its integration points; don't reinvent and miss one.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nagi-ovo/voyager](https://github.com/Nagi-ovo/voyager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
