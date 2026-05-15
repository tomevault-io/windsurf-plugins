---
trigger: always_on
description: Audimo is a **native desktop application** (macOS + Windows) that
---

# Audimo — Project Context for Codex

## Product shape

Audimo is a **native desktop application** (macOS + Windows) that
optionally lets the user expose the same instance as a **web app for
remote access from their phone**. The user runs everything on their own
hardware. There is no central Audimo server, no SaaS, no multi-tenant
backend that Anthropic-or-anyone-else operates.

This is **single-user, multi-device**, not multi-tenant. Mental model:
"Plex for music + audiobooks." One user. One library. Multiple devices
(desktop = primary; phone web view = secondary, accessed remotely over
the internet to the user's own home machine).

## What this means for design decisions

1. **`AUDIMO_HOSTED=1` is a niche escape hatch, not the production
   mode.** It exists for someone who wants to spin up a public hosted
   audimo_aio addon and disable libtorrent for legal-defensibility.
   It is not the default path. The default is: user runs the desktop
   app, the addon process is bundled alongside, libtorrent is enabled,
   Real-Debrid is optional.

2. **libtorrent is a first-class playback path**, not a fallback or a
   dev-only nicety. Reliability fixes for libtorrent (port mapping, DHT
   warm-up, tracker pool, metadata timeout, peer-discovery settings,
   private-tracker guards) directly affect end users.

3. **The bundled addon is co-located with the desktop app.** Not a
   separate service the user has to install. Likely shipped as a sidecar
   binary launched by the desktop wrapper (Tauri/Electron). Bridge
   networking + explicit port maps remain the right call so multiple
   addons can coexist.

4. **Desktop ↔ mobile sync is the real "multi-device" problem.** The
   phone's web view runs in a browser without the user's desktop
   localStorage. Addon URLs (which carry secrets in path segments) need
   to move between devices. Today this works via the Export/Import flow
   in `AddonsView`. Future: QR-code seeding, or an authenticated
   "/api/addons/seed" endpoint that the mobile web app calls once on
   first launch using a desktop-issued one-time token. Never put addon
   URLs through any third-party server.

5. **Remote access security.** When the user enables the web view, the
   backend becomes internet-reachable. Auth is mandatory: API key (or
   stronger) on every endpoint. HTTPS required (recommend the user use
   Tailscale, Cloudflare Audimo, or similar — never expose a raw port to
   the public internet). Rate-limiting on auth endpoints to slow brute
   force. The API key lives on the desktop; mobile gets a derived
   per-device token.

6. **Device-as-client is the target architecture (not yet reality).**
   The goal is for each device (desktop, phone) to keep its own addon
   list in localStorage and talk to addons directly from the browser
   (addons serving `Access-Control-Allow-Origin: *`), so the FastAPI
   backend never has to hold addon URLs or credentials.

   **Current state**: backend `addons` and `addon_settings` tables
   (`backend/database.py:160`, `:172`) DO store per-user addon URLs
   and credentials, and `backend/main.py` calls addons directly
   (`_consume_addon_stream`, `cache_resolve`, search dispatch). The
   frontend client in `frontend/src/addons/` exists alongside this
   server-side path. Don't add new server-side addon-calling code;
   prefer routing through the browser. Removing the server-side path
   is a planned refactor.

7. **Legal-defensibility framing still matters.** Audimo core ships
   zero indexer / RD / torrent code. All of that lives in the optional
   `audimo_aio` addon. Users who want a clean install never see
   it. Don't propose moving any of it back into core.

8. **Single-user assumption simplifies a lot.** No per-user data
   isolation, no quota enforcement, no abuse mitigation in the addon.
   Rate-limiting is mostly there to protect upstream APIs (RD,
   Prowlarr) from runaway bugs, not to police users.

## Local environment

- **Project root**: `/Users/jordansimpson/dev/tunnel` (was `~/Downloads`
  but moved out because macOS TCC blocks launchd from reading
  `~/Downloads`).
- **Addons live in their own repos** under `github.com/audimo-addons/*`,
  cloned as siblings to this repo:
    `~/dev/audimo-aio/`, `~/dev/audimo-indexers/`, `~/dev/audimo-slskd/`,
    `~/dev/audimo-streamers/`, `~/dev/audimo-catalog/`.
  Each runs natively on its own port (aio 9004, indexers 9005, slskd 9007,
  streamers 9006). Restart after editing an addon's `server.py`:
    `launchctl kickstart -k gui/$UID/com.audimo.<short>`
  Each addon repo carries its own `*.plist.template` next to its source.
- **Install / refresh the streaming sidecar's launchd plist**:
  `scripts/install_launchd.sh streaming` renders
  `streaming_server/com.audimo.streaming.plist.template` into
  `~/Library/LaunchAgents/`. Pair: `scripts/uninstall_launchd.sh`.
  Addon plists are managed inside their respective addon repos, not
  by this script.
- **Docker compose for hosted mode** lives in each addon's repo (e.g.
  `~/dev/audimo-indexers/deploy/docker-compose.yml` for the niche
  `AUDIMO_HOSTED=1` case). Don't use it for local dev.

## Architecture quick-reference

- `frontend/` — React + Vite. The same bundle is loaded by the desktop

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [audaro/audimo](https://github.com/audaro/audimo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
