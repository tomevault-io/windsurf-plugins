---
trigger: always_on
description: **Generated:** 2026-05-05
---

# NOCTURNE-CONNECTOR — RASPBERRY PI Wi-Fi BRIDGE + SETUP UI

**Generated:** 2026-05-05
**Runtime (server):** Bun + TypeScript + Elysia
**Runtime (UI):** Vite + React 19 + Tailwind 4 + Radix UI
**Image:** Alpine-based, built via `raspi-alpine` builder
**Related repos:** `nocturned` (the daemon on the Car Thing this bridges Spotify auth + BT to), `nocturne-ui` (the kiosk UI on the Car Thing — independent), `nocturne-image` (firmware that pairs with this device).

## OVERVIEW

A flashable Raspberry Pi OS image (~60 MB) that gives the Car Thing internet by USB-tethering Wi-Fi from the Pi, while running a local web UI at `http://nocturne-connector.local` for:

1. **Initial setup wizard** (Wi-Fi, account login, BT pair the Car Thing)
2. **Spotify account linking** (OAuth callback handling)
3. **Bridge RPC** between phone/cloud services and the Car Thing

The Pi acts as a controller box; the Car Thing remains the primary user-facing device.

## STRUCTURE

```
nocturne-connector/
├── README.md
├── LICENSE                       # Apache (NOT GPL — separate from firmware repos)
├── Justfile                      # connector-api, run, lint, docker-qemu
├── build.sh                      # Top-level builder — produces output/*.img.gz
├── wpa_supplicant.conf           # Default Wi-Fi config baked into image (user edits before flash)
├── scripts/
├── resources/                    # Image-build assets
├── cache/                        # Build cache (gitignored)
├── output/                       # Final flashable image: nocturne-connector_v<X>.img.gz + .sha256
└── src/                          # The actual Bun application — gets baked into the Pi rootfs
    ├── package.json              # bun + vite + elysia + react + dbus-next + supabase-js
    ├── bunfig.toml
    ├── tsconfig.json
    ├── vite.config.ts            # Frontend build → src/dist/client/
    ├── components.json           # Radix UI / shadcn config
    ├── server/
    │   ├── index.ts              # Elysia entrypoint: REST /api/* + WS /ws (topic "events")
    │   ├── nocturne-manager.ts   # CORE orchestrator — RPC bridge + event broadcast
    │   ├── routes/
    │   │   ├── auth.ts           # /api/auth — Supabase OTP, sign-in, callbacks
    │   │   ├── spotify.ts        # /api/spotify — link/disconnect Spotify accounts
    │   │   ├── info.ts           # /api/info — version, OS
    │   │   ├── setup.ts          # /api/setup — onboarding state machine
    │   │   └── power.ts          # /api/power — reboot / shutdown
    │   ├── services/
    │   │   ├── auth-service.ts
    │   │   ├── spotify-service.ts
    │   │   ├── bluetooth-service.ts   # dbus-next → BlueZ (BT pair the Car Thing)
    │   │   ├── ota-service.ts
    │   │   └── analytics-service.ts
    │   ├── rpc/                  # Type-safe RPC client/server primitives
    │   └── utils/
    │       ├── logger.ts
    │       ├── encryption.ts
    │       └── version.ts
    └── client/                   # React UI (built into src/dist/client/)
        ├── main.tsx
        ├── App.tsx               # Router + theme provider
        ├── pages/                # SetupWizard.tsx, Dashboard.tsx, SpotifyAuth.tsx, ...
        ├── components/           # Layout.tsx, ThemeProvider.tsx, ui/* (Radix wrappers)
        └── hooks/                # useAuth, useWebSocket, ...
```

## SERVER ↔ CLIENT WIRE CONTRACT

| Surface          | Protocol                | Notes                                                                         |
| ---------------- | ----------------------- | ----------------------------------------------------------------------------- |
| `/api/*`         | HTTP REST (JSON)        | Elysia routes; auth/spotify/info/setup/power                                  |
| `/ws`            | WebSocket               | Single broadcast topic `events`; clients subscribe and receive everything    |
| (server-internal)| RPC over WebSocket      | `nocturne-manager.ts` exposes `onCall(method, handler)` + `onEvent(name, h)` for the BT-tethered Car Thing to call into |

The browser UI talks to the server only via REST + WS; it does NOT implement the RPC layer (that's used between this Pi and the Car Thing).

## CONVENTIONS

- **`src/` IS the deployable.** The image-build process drops `src/` (post-`bun run build`) into the Alpine rootfs at a fixed path, then uses `init` to `bun run start` it on boot.
- **Frontend is built ahead of time.** Vite outputs to `src/dist/client/`, served as static files by Elysia (`@elysiajs/static`). No SSR.
- **Logger is the only sanctioned log call.** `src/server/utils/logger.ts` (level-based: debug/info/warn/error). Avoid bare `console.log` in `src/server/**`.
- **DBus is the BT API**, not bluez-tools/bluetoothctl shell-outs. `dbus-next` in `bluetooth-service.ts`.
- **State persists in Supabase** (`@supabase/supabase-js`), not local SQLite — multi-device users expect their Pi state in the cloud.
- **Encryption helpers live in one place** (`server/utils/encryption.ts`); don't roll your own.

## ANTI-PATTERNS

- **Don't commit `wpa_supplicant.conf` with real credentials.** The repo file ships with placeholders; users fill it in on the SD card after flash.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [usenocturne/nocturne-connector](https://github.com/usenocturne/nocturne-connector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
