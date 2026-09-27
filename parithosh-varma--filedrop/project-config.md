---
trigger: always_on
description: Browser-to-browser P2P file transfer. Static-only: no backend, no R2/KV/Functions, no build step, no package.json, no tests/lint.
---

# FileDrop — AGENTS.md

Browser-to-browser P2P file transfer. Static-only: no backend, no R2/KV/Functions, no build step, no package.json, no tests/lint.

## Structure

- `public/index.html` — Send (dropzone + ticket) / Receive views, tab switch. No inline scripts (CSP `script-src 'self'`); UI helpers in `ui.js`.
- `public/app.js` — all transfer logic (IIFE, vanilla JS): PeerJS signaling, 4 parallel reliable DataChannels, ~250KB raw-binary framed chunks (`[fi:uint16][i:uint32][payload]`, SCTP-size negotiated), 4-deep read prefetch via `Blob.arrayBuffer()`, event-driven backpressure (2MB/conn, 8MB total), throttled progress UI (150ms), OPFS streaming receive with Blob fallback. Protocol v3 only (`filedrop-v3-`, 12-char crypto codes; v2 6-char codes are rejected with a re-share prompt). SAS verify (`showSas`/`hideSas` → `send-sas`/`receive-sas`) + ephemeral vault opt-out (`vault-off`). Same number both tabs: sender meta carries grand-total `tb`; receiver divides stored/`tb` and reports `{t:"progress", pct, done, total}` ~4/s (capped 99 until truly done, final 100 just before `{t:"received"}`); sender mirrors while fresh (2s) else falls back to bytes-pushed. Sender sorts each batch smallest-first (stable; ties keep drop order) so cheap files complete early. Sender auto-zip: 6+ files (any size) OR 2+ files ≥1 GiB total (and zip fitting 2 GiB `MAX_FILE_SIZE`, zip32 32-bit fields, `MAX_TOTAL_BYTES` minus headroom) pack into one STORE `.zip` (`filedrop-<code>.zip`, real CRCs, no data descriptors) via `zipShouldPack`/`packZipFiles`/`makeZipFacade` (`startZippedSend`); receiver needs no changes. Smaller batches send file-by-file as before.
- `public/ui.js` — resend affordance, tab aria-pressed sync, receive progress mirroring (extracted for CSP).
- `public/vendor/` — vendored `peerjs.min.js` (1.5.4), `qrcode.min.js` (local QR, no third-party leak), `fonts/` (self-hosted Space Grotesk + Space Mono latin woff2 — no Google Fonts requests), animation libs. No runtime CDN.
- `public/_headers` — Cloudflare security headers: strict self-only CSP (`script-src 'self'` — no `unsafe-inline`/`unsafe-eval`, no third-party script/img/connect sources such as ads/trackers; ad scripts get full DOM/IDB access to codes + file bytes and break the static-only trust model — never re-add without a security review), `frame-ancestors 'none'`, HSTS, `X-Frame-Options: DENY`, `Referrer-Policy: no-referrer`, `Permissions-Policy` (camera/mic/geo/payment/usb/display-capture/bluetooth/hid/serial/fullscreen), COOP/CORP. `index.html` carries a matching `<meta http-equiv=CSP>` fallback for `python http.server` previews (minus `frame-ancestors`, which is header-only) — BOTH policies enforce conjunctively, so any CSP change must be mirrored in both files or the stricter one silently blocks the resource. Per-script SRI `integrity="sha384-…"` on local scripts — recompute hashes after ANY JS change (see deploy check below).
- `public/styles.css` — airmail counter aesthetic (par-avion `.stripe` + perforated `.ticket`); self-hosted Space Grotesk + Space Mono via `@font-face`; `.filewarn` risky-type badge; `.sas` verify + `.ephem` opt-out; fx-ambient/confetti additive.
- `wrangler.jsonc` — Worker name `filedrop`, assets `./public`
- `.wrangler/` is gitignored build/deploy output — never commit

## Commands

Local preview (no install):

```bash
python3 -m http.server -d public 8000
# open http://localhost:8000 — use two tabs/browsers to test send/receive
```

Deploy (project must already exist for classic Pages):

```bash
npx wrangler deploy
npx wrangler pages deploy ./public --project-name=fdrop --force
```

No build, no test suite. Verify manually: send 1 file + batch in tab A, receive via code and via `?code=XXXXXX` link in tab B, confirm per-file download links + 100% state. Check `≤520px` width (ticket stacks, linkrow becomes column). After ANY JS edit: `node --check public/app.js` + recompute SRI (`python3 -c` sha384 over the 9 scripts) and update `index.html` integrity attrs, or browsers will block the changed file.

## Conventions / gotchas

- Static-only, no runtime CDN: use `vendor/` files. Never re-add `unpkg`/`api.qrserver.com` (code leak + supply-chain), Google Fonts links, or ad/tracker scripts (they widen trusted JS with full DOM/IDB access to codes + file bytes). QR via `renderQrLocal()` only; fonts via `vendor/fonts` + `@font-face` only.
- Signaling: public PeerJS cloud (`0.peerjs.com`); bytes are P2P only. Sender uses `filedrop-v3-<12-char>` (`crypto.getRandomValues`, `ABCDEFGHJKMNPQRSTUVWXYZ23456789`); v2 6-char codes are rejected (`submitReceive` requires exactly 12). Codes expire after 30 min (`CODE_TTL_MS`), single-receiver binding while live (`sendOwnerPeer`), receive rate-limit 5/min with generic errors (no oracle). SAS verify: both sides show the same 6-digit `showSas()` code derived from `code|sender|receiver` — users compare over a second channel to catch a signaling MITM.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Parithosh-Varma/filedrop](https://github.com/Parithosh-Varma/filedrop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
