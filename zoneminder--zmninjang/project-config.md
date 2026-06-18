---
trigger: always_on
description: Handles platform differences (Capacitor HTTP on mobile, fetch on web), logging, and authentication automatically.
---

# Development Guidelines

## Rules

These are non-negotiable. Every rule applies to all communication: responses, commits, docs, code comments.

1. **Plain, factual writing**. Applies everywhere: responses, commits, code comments, docs, PR bodies, issue descriptions.
   - **Banned superlatives and marketing speak**: comprehensive, critical, major, robust, powerful, extensively, thoroughly, excellent, amazing, significant, seamless, intuitive, user-friendly, modern, cutting-edge, state-of-the-art, best-in-class, ground-up rewrite, world-class, blazingly fast.
   - **Banned AI slop and storytelling**: "Let's...", "As you can see", "It's important to note", "Imagine you have...", "In the real world", "Key Takeaways" or "Summary" sections that restate what was already said, multi-paragraph "why we did it this way" essays.
   - **Banned hand-wavy claims**: "designed to scale", "built for the modern web", "production-ready". State the specific fact (e.g. "handles 50k events/min on a Pi 4") or cut the claim.
   - **No em-dashes** (—). Use a period, comma, colon, or rephrase. Example: replace "Token refresh runs every 60s — checks expiry and refreshes if within leeway" with "Token refresh runs every 60s. It checks expiry and refreshes if within leeway."
   - First-person honesty is fine ("this was primarily to educate me as I did not have React experience"). Don't sand it off.
2. **Issues first**: create a GitHub issue before implementing features or fixing bugs. If an issue already exists, refer to it. 
3. **Test first, verify before commit**: write tests first, run `npm test` + `tsc --noEmit` + `npm run build` + relevant e2e tests before every commit. Build passing is not proof code works. **Always run `npm run build` (not just `tsc --noEmit`) as the final check**: `tsc -b` used by the build catches stricter errors (unused variables, type narrowing) that `tsc --noEmit` misses. Never commit if the build fails.
4. **Update docs**: update `docs/developer-guide/` in the same session when adding new APIs, components, utilities, or hooks and/or `docs/user-guide` for changed/updated or new functionality
5. **i18n all languages**: never hardcode user-facing strings. Update ALL translation files: en, de, es, fr, zh.
6. **Cross-platform**: test on iOS, Android, Electron desktop, phone portrait + landscape. Device e2e tests (`ios-phone`, `android`, etc.) are manual-invoke-only. Only `npm run test:e2e` (web) runs in the automated workflow.
7. **Profile-scoped settings**: read/write via `getProfileSettings`/`updateProfileSettings`. Never use global singletons.
8. **Bandwidth settings**: all polling/refresh features must use `useBandwidthSettings()` (or `getBandwidthSettings()` outside React). Never hardcode polling intervals.
9. **Logging**: use `log.*` component helpers with explicit LogLevel, never `console.*`. See `lib/logger.ts` for available helpers.
10. **HTTP**: use `lib/http.ts` abstractions (`httpGet`, `httpPost`, etc.), never raw `fetch()` or `axios`.
11. **Text overflow**: use `truncate` + `min-w-0` in flex containers; add `title` for tooltips. Multi-line: `line-clamp-N`.
12. **Small files**: DRY, ~400 LOC max, extract complex logic to separate modules.
13. **`data-testid`**: add `data-testid="kebab-case-name"` to all interactive elements. Required for e2e tests.
14. **Capacitor plugins**: dynamic imports only with platform checks. Never static imports. Match `@capacitor/core` major version. Add mock to `tests/setup.ts`.
15. **Mobile downloads**: use CapacitorHttp base64 directly. Never convert to Blob on mobile (OOM risk).
16. **No plan files in git**: delete `.md` plan files once the feature is complete.
17. **Complete features fully**: don't leave features half-implemented.
18. **User approval before merge**: never merge to main without user approval.
19. **One logical change per commit**: use conventional format: `feat:`, `fix:`, `docs:`, `test:`, `chore:`, `refactor:`. Reference issues with `refs #<id>` or `fixes #<id>`.
20. **Don't batch unrelated changes**: split into separate commits.
21. **Analyze test failures**: read error output and fix systematically. Don't retry blindly.
22. **Concise i18n labels**: button, tab, and action labels must be short across all languages. Prefer single-word synonyms (ES: "Ajustes" not "Configuración", DE: "Speichern" not "Änderungen speichern", FR: "Enregistrer" not "Enregistrer les modifications"). Test translations fit on a 320px-wide phone screen. Add `min-w-0` + `truncate` to flex containers with translated button text as a safety net.
23. **Date/time formatting**: all user-facing date/time display must use `useDateTimeFormat()` hook (or `formatAppDate`/`formatAppTime`/`formatAppDateTime` from `lib/format-date-time.ts` outside React). Never hardcode date-fns `format()` with literal patterns for user-visible output. This includes canvas rendering, tooltips, labels, and scrubber overlays.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZoneMinder/zmNinjaNg](https://github.com/ZoneMinder/zmNinjaNg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
