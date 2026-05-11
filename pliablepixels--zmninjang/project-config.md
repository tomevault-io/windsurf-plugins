---
trigger: always_on
description: Handles platform differences (Capacitor HTTP on mobile, fetch on web), logging, and authentication automatically.
---

# Development Guidelines

## Rules

These are non-negotiable. Every rule applies to all communication: responses, commits, docs, code comments.

1. **No superlatives** — never use "comprehensive", "critical", "major", "robust", "powerful", "extensively", "thoroughly", "excellent", "amazing", "significant", etc. Plain, factual language only.
2. **Issues first** — create a GitHub issue before implementing features or fixing bugs. Commit directly to main only for docs-only changes, refactors without behavior change, test additions for existing code, or dependency updates.
3. **Test first, verify before commit** — write tests first, run `npm test` + `tsc --noEmit` + `npm run build` + relevant e2e tests before every commit. Build passing is not proof code works. **Always run `npm run build` (not just `tsc --noEmit`) as the final check** — `tsc -b` used by the build catches stricter errors (unused variables, type narrowing) that `tsc --noEmit` misses. Never commit if the build fails.
4. **Update docs** — update `docs/developer-guide/` in the same session when adding new APIs, components, utilities, or hooks and/or `docs/user-gudie` for changed/updated or new functionality
5. **i18n all languages** — never hardcode user-facing strings. Update ALL translation files: en, de, es, fr, zh.
6. **Cross-platform** — test on iOS, Android, Desktop, phone portrait + landscape. Device e2e tests (`ios-phone`, `android`, etc.) are manual-invoke-only — only `npm run test:e2e` (web) runs in the automated workflow.
7. **Profile-scoped settings** — read/write via `getProfileSettings`/`updateProfileSettings`. Never use global singletons.
8. **Bandwidth settings** — all polling/refresh features must use `useBandwidthSettings()` (or `getBandwidthSettings()` outside React). Never hardcode polling intervals.
9. **Logging** — use `log.*` component helpers with explicit LogLevel, never `console.*`. See `lib/logger.ts` for available helpers.
10. **HTTP** — use `lib/http.ts` abstractions (`httpGet`, `httpPost`, etc.), never raw `fetch()` or `axios`.
11. **Text overflow** — use `truncate` + `min-w-0` in flex containers; add `title` for tooltips. Multi-line: `line-clamp-N`.
12. **Small files** — DRY, ~400 LOC max, extract complex logic to separate modules.
13. **`data-testid`** — add `data-testid="kebab-case-name"` to all interactive elements. Required for e2e tests.
14. **Capacitor plugins** — dynamic imports only with platform checks. Never static imports. Match `@capacitor/core` major version. Add mock to `tests/setup.ts`.
15. **Mobile downloads** — use CapacitorHttp base64 directly. Never convert to Blob on mobile (OOM risk).
16. **Tauri packages** — JS `@tauri-apps/*` and Rust `tauri-plugin-*` versions must match. Update `package.json` and `Cargo.toml` together.
17. **No plan files in git** — delete `.md` plan files once the feature is complete.
18. **Complete features fully** — don't leave features half-implemented.
19. **User approval before merge** — never merge to main without user approval.
20. **One logical change per commit** — use conventional format: `feat:`, `fix:`, `docs:`, `test:`, `chore:`, `refactor:`. Reference issues with `refs #<id>` or `fixes #<id>`.
21. **Don't batch unrelated changes** — split into separate commits.
22. **Analyze test failures** — read error output and fix systematically. Don't retry blindly.
23. **Concise i18n labels** — button, tab, and action labels must be short across all languages. Prefer single-word synonyms (ES: "Ajustes" not "Configuración", DE: "Speichern" not "Änderungen speichern", FR: "Enregistrer" not "Enregistrer les modifications"). Test translations fit on a 320px-wide phone screen. Add `min-w-0` + `truncate` to flex containers with translated button text as a safety net.
24. **Date/time formatting** — all user-facing date/time display must use `useDateTimeFormat()` hook (or `formatAppDate`/`formatAppTime`/`formatAppDateTime` from `lib/format-date-time.ts` outside React). Never hardcode date-fns `format()` with literal patterns for user-visible output. This includes canvas rendering, tooltips, labels, and scrubber overlays.
25. **Self-updating rules** — when the user gives guidance that establishes a general pattern (e.g., "all X should use Y"), check whether it belongs as a persistent rule in this file. If so, add it here so future sessions follow it automatically.

---

## Working Directory

All `npm` commands must be run from the `app/` directory.

Structure:
- `./` — workspace root (AGENTS.md, docs/, scripts/)
- `app/` — main application (run npm commands here)
- `app/src/` — source code
- `app/tests/` — e2e test features and helpers

---

## Testing

### Philosophy: Be a Human Tester
Every test must verify what a real human would verify: "Can I accomplish this task? Does this look right? Does the data make sense?"

- Verify outcomes (data changed, navigation happened, file downloaded) — not just element presence
- Fill forms and verify data persists after refresh or navigation
- Test error states, edge cases, and device-specific layout behavior
- Add `@visual` screenshots to catch layout regressions
- Never write "check heading is visible" as a test — that's not testing anything
- Never mock the thing you're testing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pliablepixels/zmNinjaNg](https://github.com/pliablepixels/zmNinjaNg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
