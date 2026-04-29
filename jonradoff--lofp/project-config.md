---
trigger: always_on
description: Resurrecting a 1990s MUD from original script files. The original game engine source code is lost; only the content scripts and documentation survive. We reverse-engineer a working game from those.
---

# LoFP — Legends of Future Past

Resurrecting a 1990s MUD from original script files. The original game engine source code is lost; only the content scripts and documentation survive. We reverse-engineer a working game from those.

## Architecture

- **Backend**: Go + gorilla/mux + MongoDB at `engine/`
- **Frontend**: React 19 + TypeScript + Vite + Tailwind 4 at `frontend/`
- **Original Scripts**: `original/scripts/` (read-only reference, 333 .SCR files)
- Frontend dev server: port 4992, Backend server: port 4993
- Start both: `./start.sh`

## Build Check

```sh
cd engine && go build ./...
cd frontend && npx tsc --noEmit
```

## Reference Documentation

These original documents are the primary sources for game mechanics and world data:

| Document | Path | Description |
|----------|------|-------------|
| GM Manual | `original/GM Pages/MANUAL.DOC` | Comprehensive GM reference: combat, stats, items, monsters, skills, XP tables, all GM commands |
| GM Script Guide | `original/GMSCRIPT.DOC` | Script language reference: room/item/monster definitions, IFVERB/IFENTRY/IFSAY blocks, variables |
| Player Manual | `original/legends/LEGENDS.DOC` | Player-facing docs: commands, spells, psionics, skills, races, crafting |
| Session Capture | `original/legends/shirla.cap` | 1996 gameplay session capture — invaluable for combat output format, spell interactions, original message text |
| Script Config | `original/scripts/LEGENDS.CFG` | Master config listing all .SCR files to load in order |
| GM Pages | `original/GM Pages/` | Additional GM reference materials (various .DOC files) |

When implementing new features, always cross-reference these documents and the session capture to match original behavior.

## Multi-Machine Coordination

Production runs multiple Fly.io machines. ALL mutable world state must be coordinated via the MongoDB-backed hub (`engine/internal/hub/`):

- **Messages**: broadcasts, whispers, global announcements → published to `events` collection, delivered via Change Streams
- **Player presence**: WHO list, room occupancy → `presence` collection with TTL heartbeat
- **Room state changes**: item open/close/lock, item drops/pickups, script mutations (vals, itembits) → `room_state_change` events via hub
- **Any new mutable state** added to rooms, items, or global data MUST call `notifyRoomChange()` or publish through the hub, or players on different machines will see inconsistent worlds

## After Server Changes

After making changes to the backend (engine/), restart the Go server:
```sh
kill $(lsof -ti:4993) 2>/dev/null; sleep 1; cd engine && go run cmd/lofp/main.go &
```
Load .env first if needed: `source .env`

## Deploying to Production

**CRITICAL: The Google Client ID MUST be passed as a build arg or Google login will break.**

The frontend uses `VITE_GOOGLE_CLIENT_ID` at build time (baked into the JS bundle by Vite). If it's missing, the "Sign in with Google" button disappears and users see "Authentication Not Configured".

### Deploy command (always use this exact pattern):
```sh
GCID=$(grep GOOGLE_CLIENT_ID .env | cut -d= -f2) && fly deploy --build-arg "VITE_GOOGLE_CLIENT_ID=$GCID"
```

**Why this specific pattern?** The Bash tool's shell does not persist exported variables between commands. `source .env && fly deploy --build-arg VITE_GOOGLE_CLIENT_ID=$GOOGLE_CLIENT_ID` silently passes an empty string because `source` sets but does not export the variable, and the Bash tool may run commands in separate shell contexts. Using `grep | cut` directly extracts the value inline, which is reliable.

If using `--no-cache` to force a fresh build, append it:
```sh
GCID=$(grep GOOGLE_CLIENT_ID .env | cut -d= -f2) && fly deploy --build-arg "VITE_GOOGLE_CLIENT_ID=$GCID" --no-cache
```

### After deploying, verify Google login:
```sh
fly ssh console -a lofp -C "grep -c 718491 /app/static/assets/index-*.js"
```
This should return `1`. If it returns `0`, the build arg was not passed correctly.

### Local dev vs. production:
- **Local**: `.env` file at project root contains `GOOGLE_CLIENT_ID=...` (among other secrets). Vite reads `VITE_GOOGLE_CLIENT_ID` from the environment when running `npm run dev`.
- **Production**: Fly.io secrets store `MONGODB_URI`, `JWT_SECRET`, `RESEND_API_KEY`, `SSH_HOST_KEY` (set via `fly secrets set`). The Google Client ID is NOT a Fly secret — it's a **build arg** because Vite needs it at build time, not runtime.
- **Never commit `.env`** — it contains production secrets. It is in `.gitignore`.

## MUD Client Protocols

Reference: https://wiki.mudlet.org/w/Manual:Supported_Protocols

Telnet server (`engine/internal/api/telnet.go`) implements these protocols:

### GMCP (option 201)
- **Core.Hello** — sent on connect: `{"client":"LoFP","version":"11.1.0"}`
- **Core.Supports.Set** — handled from client to track subscribed packages
- **Char.Vitals** — sent after every command: `bp`, `maxbp`, `mana`, `maxmana`, `psi`, `maxpsi`, `fatigue`, `maxfatigue`, `position`, `conditions`
- **Char.Status** — sent on login: `name`, `fullname`, `race`, `gender`, `level`, `experience`, `gold`, `silver`, `copper`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jonradoff/lofp](https://github.com/jonradoff/lofp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
