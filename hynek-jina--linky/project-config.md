---
trigger: always_on
description: Mobile-first PWA for contacts, Nostr messaging, and Lightning/Cashu payments. Local-first architecture using Evolu for offline storage and cross-device sync.
---

# Linky

Mobile-first PWA for contacts, Nostr messaging, and Lightning/Cashu payments. Local-first architecture using Evolu for offline storage and cross-device sync.

See @README.md for project overview.

## Commands

```bash
bun install                # Install dependencies
bun run dev                # Start Vite dev server
bun run build              # Production build (tsc -b && vite build)
bun run site:dev           # Start the public Linky website for linky.fit
bun run site:build         # Build the public Linky website
bun run native:android:add # Generate the Capacitor Android project once
bun run native:apk:debug   # Build the web app, sync Capacitor, assemble debug APK
bun run native:apk:release # Build the web app, sync Capacitor, and assemble a signed release APK
bun run native:aab:release # Build the web app, sync Capacitor, and bundle a signed release AAB for Play upload
bun run native:ios:add     # Generate the Capacitor iOS project once
bun run push:dev           # Start the Bun push notification service in watch mode
bun run push:start         # Start the Bun push notification service once
bun run check-code         # Run ALL checks: typecheck → eslint --fix → prettier --write
bun run typecheck          # TypeScript type checking only
bun run eslint             # Lint + autofix all workspaces
bun run prettier           # Format + autofix all workspaces
```

IMPORTANT: Always run `bun run check-code` after making changes. It runs typecheck first, then eslint and prettier which autofix what they can. If typecheck or non-autofixable eslint errors remain, fix them manually and re-run until all checks pass.

Native Android builds require Java 17. `apps/native-shell/scripts/with-java17.sh` prefers an installed macOS JDK 17 automatically before running Capacitor/Gradle commands, and `apps/native-shell/scripts/patch-android-java.sh` rewrites Capacitor-generated Android compile options from Java 21 to Java 17 after add/sync.

## Monorepo Structure

- `apps/site/` - Public marketing/landing website intended for `linky.fit`
  - includes a feature-video showcase fed from static assets in `apps/site/public/videos/`, with mobile viewport-triggered playback, desktop hover-triggered playback, and automatic once-only progression to the next clip
  - landing CTA offers Web app, Google Play, and APK targets; it defaults to Web app generally, but prefers Google Play when the browser is on an Android phone
  - landing page and `/cashu` share a top-right site menu with language + display-currency selectors and a CTA into `https://app.linky.fit`; `/cashu` uses the selected display currency for its token amount header, with `₿` still meaning sats rather than whole bitcoin
  - `/cashu` token launch now prefers the native `cashu://receive?token=...` deep link, falls back to installed PWA handling via `web+cashu://receive?token=...`, and only then falls back to the browser `https://app.linky.fit/#wallet?cashu=...` import route
  - Vercel rewrites `/.well-known/lnurlp/:user` and `/.well-known/nostr.json` into `apps/site/api/` serverless handlers so apex `linky.fit` can expose LNURL/NIP-05 while forwarding to `NPUBCASH_BASE_URL` (default `https://npub.linky.fit`); LNURL responses rewrite the returned callback back to the apex host
  - includes a dedicated `cashu/` entry page for `linky.fit/cashu`, which can ingest a token from manual paste, query string, or preferably URL hash, inspect the token client-side, present `Vyzvednout v Linky` / primary Linky-open CTA first, reveal QR-copy plus Lightning-address redeem flow behind a secondary `Další možnosti` toggle, and redeem to a Lightning address; redeem sends the maximum LN-address amount the mint+LNURL flow allows, queues anonymous payment telemetry to the same collector/feed used by the app, and forwards any leftover token value as a private Nostr gift wrap to the configured collector `npub` from a one-time sender identity instead of returning change
- `apps/web-app/` - Main React app (Vite + SWC)
  - non-essential startup network work now defers until after the first idle window while the browser is online via `src/hooks/useDeferredOnlineReady.ts`; first render should prefer local Evolu/localStorage state, with fiat-rate refresh, mint-info refresh, relay probing, push bootstrap, and Evolu server probing catching up later
  - PWA manifest registers `web+cashu` protocol handling to route installed-app launches into `/#wallet?cashu=%s`, letting the site prefer installed PWA over plain browser tabs when the native app is unavailable
  - wallet receive (`#wallet/topup`) keeps the amount-entry invoice flow, now also exposes secondary `No amount`, `Paste`, and `Scan` actions at the bottom, and `#wallet/topup/no-amount` renders a reusable QR from the user's lightning address for amount-less LNURL-pay receipts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hynek-jina/linky](https://github.com/hynek-jina/linky) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
