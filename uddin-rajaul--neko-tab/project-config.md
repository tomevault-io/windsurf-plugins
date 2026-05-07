---
trigger: always_on
description: When working with this codebase, prioritize readability over cleverness. Ask clarifying questions before making architectural changes.
---

## Core Principles
When working with this codebase, prioritize readability over cleverness. Ask clarifying questions before making architectural changes.

## Branch & PR Workflow

**Always work in a feature branch. Never commit directly to `main`.**

```bash
git checkout main && git pull origin main
git checkout -b <type>/<short-description>
# do the work
git push origin <branch-name>
# open PR on GitHub — review and merge is up to Raj
```

Branch naming:
- `feat/` — new feature
- `fix/` — bug fix
- `chore/` — cleanup, refactoring, non-functional changes
- `docs/` — documentation only

**CLAUDE.md is the only file that can be updated directly on `main`.**

PR descriptions should clearly state:
- What changed and why
- Any side effects or things to watch out for
- If it closes an issue, mention it (`Closes #N`)

---

## 1. Think Before Coding
**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First
**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked
- No abstractions for single-use code
- No "flexibility" or "configurability" that wasn't requested
- No error handling for impossible scenarios
- If you write 200 lines and it could be 50, rewrite it

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes
**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting
- Don't refactor things that aren't broken
- Match existing style, even if you'd do it differently
- If you notice unrelated dead code, mention it — don't delete it without asking

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused
- Don't remove pre-existing dead code unless explicitly asked

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution
**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

## Workflow Patterns

### Research → Plan → Code → Commit → PR
1. **Research**: Read relevant files, understand context
2. **Plan**: State approach before coding
3. **Code**: Implement on a feature branch
4. **Build**: Run `./node_modules/.bin/vite build` to verify no errors
5. **Commit**: Clear commit messages describing the change
6. **PR**: Push branch and open a PR — Raj reviews and merges

## Code Standards

- Use clear, descriptive variable and function names
- Write self-documenting code with minimal comments
- Comments should explain "why", not "what"
- Keep functions small and focused on single responsibility
- Prefer composition over inheritance
- Follow existing patterns in the codebase
- Always run a build before pushing — zero TypeScript errors required


## Project Structure

```
src/
  components/   # React components, one per file
  hooks/        # useLocalStorage, useSettings, useBookmarks, useTime, useGoogleCalendar
  utils/        # imageToAscii
  types.ts      # Shared TypeScript interfaces
  styles/       # Split CSS files, imported via index.css manifest
  index.css     # Pure @import manifest — do not add styles here directly
  App.tsx       # Root layout
public/
  manifest.json # Chrome Extension MV3 — placeholders injected at build time
  background.js # Service worker (focus mode blocking)
  dist.pem      # Extension private key — DO NOT regenerate, DO NOT commit changes
screenshots/    # Only image.png and terminal.png are used in README
```

## Build System & Manifest Injection

`vite.config.ts` runs a `closeBundle` plugin that post-processes `dist/manifest.json`:
- Replaces `__GOOGLE_CLIENT_ID__` with `GOOGLE_CLIENT_ID` from `.env.local`
- Replaces `__GOOGLE_EXTENSION_KEY__` with `GOOGLE_EXTENSION_KEY` from `.env.local`
- If `GOOGLE_EXTENSION_KEY` is missing or invalid base64, the `key` field is stripped entirely (safe for local dev without OAuth)

`.env.local` holds both values and is gitignored. Never hardcode client IDs in source.

## Chrome Extension — MV3 Rules

These are hard constraints. Violating them causes silent failures and extension instability:

**CSP (Content Security Policy)**
- `manifest.json` must declare `content_security_policy.extension_pages` explicitly
- Inline `<style>` blocks in `index.html` require their SHA-256 hash in the CSP `style-src`
- When adding or changing the inline style block, get the hash from the Chrome extension error console and add it: `'sha256-XXXX='`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uddin-rajaul/Neko-Tab](https://github.com/uddin-rajaul/Neko-Tab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
