---
trigger: always_on
description: Concise rules for building accessible, fast, delightful UIs. Use MUST/SHOULD/NEVER to guide decisions.
---

Concise rules for building accessible, fast, delightful UIs. Use MUST/SHOULD/NEVER to guide decisions.

## Code Style

- NEVER: Add inline or block code comments. Let names and structure carry intent; the code should read on its own.
- SHOULD: Rename or restructure instead of annotating when something is unclear.

## Architecture & Modules

- MUST: Product pages are thin. `src/pages/<name>.js` keeps only imports, `FEATURES`, `REPOS`, the exported `Head`, and the page component. Sections live one-per-file in `src/components/pages/<name>/` with a `shared.js` for cross-section primitives. That layout (PR #2153) is what lets parallel sessions work without colliding — follow it for new pages.
- MUST: Keep every React component under ~300 lines (react-doctor CI gate). Decompose the way the `hero/` directories do: `index.js` owns state/handlers/composition, effects live in `use-*.js` hooks with their bodies intact, JSX regions become presentational components whose prop names match the original bindings.
- MUST: `const REPOS = [...]` and `repos={REPOS}` stay literally in the page file — `test/components/open-source-pattern.js` regex-parses page source. `test/components/home-hero-*.js` also read source; when moving code, repoint those tests, never weaken them.
- MUST: Reuse the shared patterns instead of copying: `patterns/LiveTiming` (live stat display), `patterns/SpeedSection` (competitor benchmark section, copy/data as props), `hook/use-demo-ui` (attract/focus flag reducer), `hook/use-error-modal-focus` (dialog focus trap/Escape/restore). Each existed as 4-7 diverging copies before being unified.
- NEVER: Rewrite a working idiom to satisfy a lint matcher. `doctor.config.json` is the ledger of verified false positives — suppress there with evidence in the commit message. When decomposing files, repoint any suppression pinned to the old path.
- NEVER: Delete an "unused" file or export on a dead-code scanner's word alone. MDX content imports (`src/content/**/*.md` has real `import` statements), Gatsby conventions (`src/html.js`, GraphQL fragment spreading) and `scripts/` usage are invisible to it — grep those first.

## Demo Correctness

The interactive demos (hero omniboxes, capabilities panes) share one contract:

- MUST: Blur handlers read `e.target.value` at blur time and compare against the last submitted URL (`navStack[navIndex]`), not against live input state (which `handleChange` keeps synced, making the comparison always false). Enter/Escape/back/forward/history selection set `skipBlurRef` before triggering blur.
- MUST: Fetch hooks reset loading in `finally`, gated by that invocation's own abort signal — a superseded request must never clear the newer request's spinner. Deferred-clear flags (img/PdfViewer `onLoad` owns the reset) compose with this, not replace it.
- MUST: `encodeURIComponent` any user URL interpolated into displayed or copied API URLs (the fetch hooks already encode; the display strings drifted).
- MUST: Dropdown/listbox items use `onMouseDown={e => e.preventDefault()}` (protects against the blur race) with the action in `onClick` (fires for both mouse and keyboard). `onMouseDown`-only items are dead to keyboard users.
- MUST: Attract loops guard every setter after an `await` with the loop's `cancelled`/`check()` pattern, and their frozen first-render closures are deliberate — dep arrays list the destructured stable values (setters, refs, stable callbacks), never the caller's object bag.

### Tokenized Styling Syntax

- MUST: Use scalable design tokens from `src/theme/index.js` for UI styling.
- MUST: Avoid hardcoded values when a design token already exists.
- MUST: Avoid extending design tokens; use the existing token set.
- MUST: Use `theme({...})` for any property supported by styled-system before using raw CSS declarations.
- MUST: Treat every styled-system-supported property (see styled-system's prop groups and `src/theme/index.js`) as a `theme({...})`-first property, aliases included.
- MUST: Decompose raw CSS declarations into styled-system keys whenever possible (for example use `py`/`px` instead of `padding`, `mt`/`mb` instead of `margin`, and `borderBottom` + `borderBottomColor` instead of raw `border-bottom`).
- MUST: For border tokens, prefer tokenized border props over string interpolation: `borderBottom: 1` + `borderBottomColor: 'black05'` (or `borderColor` if all sides share the same color).
- MUST: Consolidate related tokenized properties into a single `theme({...})` call per selector block (and per media block), instead of multiple adjacent `theme(...)` calls.
- MUST: Prefer responsive arrays/objects for styled-system props inside one `theme({...})` call (for example `p: [3, 3, 4, 4]`) instead of token-only media-query overrides.
- MUST: Prefer semantic token references in `theme({...})`, such as `fontFamily: 'mono'`, `fontWeight: 'bold'`, `color: 'black'`, `fontSize: 1`, `lineHeight: 0`, `letterSpacing: 0`.
- MUST: Apply the same rule inside styled components and inline `css={theme({...})}` objects.
- NEVER: Use raw token interpolation (for example `font-size: ${fontSizes[0]}`) when the same style can be expressed via `theme({...})`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microlinkhq/www](https://github.com/microlinkhq/www) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
