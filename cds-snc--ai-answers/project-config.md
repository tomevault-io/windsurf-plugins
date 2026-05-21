---
trigger: always_on
description: - **React build restriction**: Files imported by frontend code (`src/`) must live inside `src/`. Never place shared config intended for UI components in `config/` (root) — use `src/config/` instead. Server-side code (`api/`, `agents/`, `services/`) can import from anywhere.
---

# Coding Agent Instructions

## Environment notes
- **React build restriction**: Files imported by frontend code (`src/`) must live inside `src/`. Never place shared config intended for UI components in `config/` (root) — use `src/config/` instead. Server-side code (`api/`, `agents/`, `services/`) can import from anywhere.
- **Test runner**: This project uses **vitest**, not jest. Run tests with `npx vitest run <path>` (or `npm test` for all).
- **CSS loading**: All app styles are loaded once in `src/App.js` (`global.css`, `admin.css`, `chat.css`). Never import these files in individual pages or components — they are already globally available. Do not move these imports to `index.js` either: `App.js` must load after `index.js`'s GCDS CSS (`gcds-utility.min.css` imports Lato/Noto Sans from Google Fonts) so that webpack resolves the stylesheets in the correct order. Moving app CSS into `index.js` alongside GCDS CSS breaks the GC Design System fonts.

## How to work well in this codebase

1. **State assumptions early.** Before implementing anything non-trivial, say what you're assuming so we can catch misalignment before code is written.
2. **Pause on ambiguity.** If you hit inconsistencies, conflicting requirements, or unclear specs, surface the tradeoff or ask for clarification rather than guessing.
3. **Push back when it helps.** If the human's approach has clear problems, point it out directly and propose an alternative. Agreeing to avoid friction wastes everyone's time.
4. **Keep it simple.** Favour the boring, obvious solution. If 100 lines would do and you wrote 1000, something went wrong.
5. **Stay scoped.** Avoid removing comments you don't understand, "cleaning up" code orthogonal to the task, refactoring adjacent systems as side effects, or deleting code that seems unused without asking first.
6. **Flag dead code.** After refactoring or implementing changes, point out code that's now unreachable and ask what to do with it.
7. **Clarify success criteria.** If instructions don't include them, reframe the goal explicitly so you can loop, retry, and problem-solve rather than following steps that may not lead anywhere.
8. **Test-first for non-trivial logic.** Write the test that defines success, implement until it passes, then show both.
9. **Run existing tests after changes.** After modifying code, run the relevant test suite (`npm test` or the specific test file) to catch regressions before considering the task done.
10. **Check for downstream impact.** After changing a shared function, utility, or service, trace its callers to verify the change doesn't break other consumers. Don't assume the only usage is the one you're fixing.

## Documentation Regeneration

When any file in `agents/prompts/` is changed (except department scenarios in `agents/prompts/scenarios/context-*/`), regenerate the system prompt documentation:

```bash
node scripts/generate-system-prompt-documentation.js
```

This keeps `docs/agents-prompts/system-prompt-documentation.md` in sync with the actual prompts.

## Official languages
**English users and admins and partners must be served in English. French users and admins and partners must be served in French.** This applies to all pages and tools — public-facing, admin, and partner.

**Never hardcode user-facing text in components or pages.** All text visible to users must use translation keys via `t()` and have entries in both `src/locales/en.json` and `src/locales/fr.json`. When adding any new text (column headers, labels, buttons, messages, placeholders, error messages, status messages, option labels, etc.), always add the corresponding key to both locale files in the same PR — don't rely on the fallback string in `t('key', 'fallback')` or `t('key') || 'fallback'`.

### Exceptions
- **Backend/console/database output**: `console.log`, `console.error`, server-side log strings, developer-facing CLI output, and dynamic content retrieved from the database are exempt.
- **Internal technical identifiers used as option values**: e.g. workflow names like `GenericGraph` where the value and label are the same internal enum — these are not user-facing text.

### Sentence case
All text visible to users uses sentence case (only the first word and proper nouns capitalised). This applies to button labels, column headers, section titles, navigation links, and option labels. Examples: `"Upload file"` not `"Upload File"`, `"Processed batches"` not `"Processed Batches"`, `"Clarifying question"` not `"Clarifying Question"`.

### Locale key hygiene
After adding, removing, or renaming locale keys, run the dead key detector:

```bash
node scripts/find-dead-locale-keys.cjs
```

This reports:
1. **Dead keys** — keys in `en.json`/`fr.json` with no detected usage in `src/`
2. **Duplicate keys** — different keys with identical values (consolidation candidates)
3. **Parity gaps** — keys present in EN but missing from FR, or vice versa

Parity gaps must be fixed before merging. Dead keys and duplicates are cleaned up incrementally — fix a few per PR rather than all at once.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cds-snc/ai-answers](https://github.com/cds-snc/ai-answers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
