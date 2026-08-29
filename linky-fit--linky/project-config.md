---
trigger: always_on
description: Mobile-first PWA for contacts, Nostr messaging, and Lightning/Cashu payments. Local-first architecture using Evolu for offline storage and cross-device sync.
---

# Linky

Mobile-first PWA for contacts, Nostr messaging, and Lightning/Cashu payments. Local-first architecture using Evolu for offline storage and cross-device sync.

See @README.md for project overview.

Package manager is **Bun** (not npm/yarn/pnpm); scripts are defined in the root `package.json`. Workspace filter: `bun run --filter @linky/web-app <script>`.

IMPORTANT: Always run `bun run check-code` after making changes. It runs typecheck first, then eslint and prettier which autofix what they can. If typecheck or non-autofixable eslint errors remain, fix them manually and re-run until all checks pass.

Native Android builds require Java 17. `apps/native-shell/scripts/with-java17.sh` prefers an installed macOS JDK 17 automatically before running Capacitor/Gradle commands, and `apps/native-shell/scripts/patch-android-java.sh` rewrites Capacitor-generated Android compile options from Java 21 to Java 17 after add/sync.

## Architecture

Architectural decisions and behavioral constraints are documented in `docs/architecture.md`. Read the relevant sections there before changing app structure, data flow, persistence, or protocols.

IMPORTANT: When you make or change an architectural decision, document it in `docs/architecture.md` in the same commit — not in this file. This file only holds commands, conventions, testing, and operational gotchas.

## Code Conventions

- TypeScript strict mode with `exactOptionalPropertyTypes`
- **NEVER use `as` or `any` to cast types** - validate with a runtime type guard instead of casting
- Branded ID types from Evolu (`ContactId`, `CashuTokenId`, `MintId`, etc.) - don't use plain strings
- Components use `interface` for props, not `type`
- LocalStorage keys use `linky.` prefix (e.g., `linky.nostr_nsec`, `linky.lang`)
- Use types from libraries (e.g., Evolu, Cashu, Nostr) instead of redefining them - look up the library's exported types first
- Prefer sparse Evolu mutation payloads: omit optional fields when empty instead of writing explicit `null` (especially `cashuToken` optional columns like `rawToken`, `mint`, `unit`, `amount`, `error`)
- Plain CSS in `App.css` - no CSS-in-JS or utility framework


### Commenting the code
- If you need to add comment to a code to justify the code being overcomplicated, the code is bad and you should do it differently - unless instructed otherwise or we specifically agree on going with this implementation. Good comments do not excuse unclear code.
- Comments should not duplicate the code! The code should be self explanatory, use function names, proper code split into logical chunks
- Explain unidiomatic code in comments - keep the comments brief and to the point if you need to write it!

## Inspector events

When implementing or refactoring a meaningful operation — user-initiated actions, network/relay/mint traffic, sync, push, notable state transitions — emit an inspector event for it. Follow the `adding-inspector-events` skill (`.agents/skills/adding-inspector-events/`) for row design, correlation links, and the no-key-material rule.

## Release Versioning

[CalVer](https://calver.org) `YY.MM.MICRO`: short year, month (no leading zero), release counter starting at `1` that resets monthly — e.g. August 2026: `26.8.1`, `26.8.2`, `26.8.3`.

Exception: August 2026 accidentally shipped as `26.9.0`, so keep releasing as `26.9.MICRO` (bump `MICRO` each release) through both August and September 2026 — do not reset the counter in September. Normal scheme resumes with `26.10` in October 2026; delete this paragraph then.

## Local dev environment

- `bun run dev` starts the local service stack (`docker-compose.dev.yml`: Nostr relay :7777, Evolu relay :4001, FakeWallet mint :3338) detached, then the web app (:5173) and push service (:8787) against it; requires Docker
- `bun run dev:prod` runs the web app on :5175 against production services (no local stack needed)
- `bun run dev:services` runs just the docker stack attached (Ctrl-C stops it)
- See the "Local dev environment" section in `docs/architecture.md` for how env overrides and vite modes work

## E2E tests

Two Playwright projects in `apps/web-app/playwright.config.ts`:

- `prod-services` — the original suite. Playwright starts `vite --mode prod-services` on :5174 and the tests hit production relays/mints.
- `local-stack` — `tests/proxy-payment.spec.ts` only, against the docker stack with the app served as a **production build** on :5176. It declares no `webServer`; compose owns the app, so bring the stack up first.

```bash
# once, and again after changing app source (VITE_* values are inlined at build time)
docker compose -f docker-compose.dev.yml --profile e2e up -d --build --wait

cd apps/web-app
bunx playwright test --project=local-stack                      # run it
bunx playwright test --project=local-stack --ui                 # step through by test.step()
bunx playwright test --project=local-stack --headed             # three live browsers
bunx playwright show-trace test-results/*local-stack/trace.zip  # after the fact
bunx playwright show-report
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [linky-fit/linky](https://github.com/linky-fit/linky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
