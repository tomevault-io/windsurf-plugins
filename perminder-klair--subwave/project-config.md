---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

SUB/WAVE is a personal internet radio station: one Icecast stream, all listeners hear the same broadcast, AI DJ picks tracks and reads scripts between them. See `README.md` for the architecture diagram and rationale.

## Common commands

```bash
# --- dev (Mac smoke test) ---
cd docker && docker compose up -d        # Icecast + Liquidsoap + Controller only
cd web && npm install && npm run dev     # web UI on :7700, separate process
cd controller && npm install && npm run dev

# --- production (single-host, Caddy edge) ---
./scripts/setup.sh                       # state defaults to <repo>/state
docker compose -f docker/docker-compose.prod.yml up -d
./scripts/generate-jingles.sh            # render Piper station idents
./scripts/update.sh                      # git pull + rebuild + rolling recreate

# Common one-offs
docker compose -f docker/docker-compose.prod.yml logs -f controller
curl http://localhost:4800/api/health    # liveness via Caddy edge (prod)
```

There is no `/skip` endpoint — track-end is the only natural transition. Liquidsoap controls pacing.

**Controller source needs a rebuild, not a restart.** `controller` `COPY`s its source at build time, so `docker compose restart controller` reruns the *same baked-in code*. `liquidsoap` is different: `radio.liq` is bind-mounted (`../liquidsoap/radio.liq:/etc/liquidsoap/radio.liq:ro` in both compose files), so editing it only needs a restart — `Dockerfile.liquidsoap` itself doesn't `COPY` the script.

```bash
cd docker && docker compose up -d --build controller     # after any controller/src/** change
cd docker && docker compose restart liquidsoap           # after radio.liq edits — bind-mounted, no rebuild
cd docker && docker compose up -d --build liquidsoap     # only after Dockerfile.liquidsoap changes
```

`web` is a Next.js dev server in local mode (`npm run dev`), so it hot-reloads — no rebuild needed for UI changes during dev. Production builds the web image; treat it like the others there.

No test runner, linter, or formatter is configured.

## Architecture

Four cooperating processes with **file-based IPC** through a shared `state/` directory (mounted at `/var/sub-wave` in containers). This is the load-bearing fact about how the system works:

- **Controller → Liquidsoap**:
  - `next.txt` — controller writes one annotated track URI; Liquidsoap polls every 1.0s, drains, and `request.queue.push`es it (`liquidsoap/radio.liq`).
  - `say.txt` — controller writes a WAV path; Liquidsoap polls every 0.5s and feeds it through `voice_queue`, which is **heavy-ducked** (`smooth_add p=0.25`) over the music bus. Used for station IDs, hourly time, weather, and listener-request intros.
  - `intro.txt` — separate channel for between-track auto-DJ links. Liquidsoap polls every 0.5s and feeds it through `intro_queue`, which is **light-ducked** (`smooth_add p=0.40`) so the song that just started stays audible underneath the voice.
  - `auto.m3u` — fallback playlist the controller rewrites every `AUTO_QUEUE_REFRESH_MINUTES` (default 60) for the current mood; Liquidsoap reloads it on file change (`reload_mode="watch"`).
  - `liquidsoap_jingle_ratio.txt` / `liquidsoap_crossfade.txt` — tiny text files written by `settings.update()`. Read once at `radio.liq` startup; changes require a Liquidsoap restart (which the controller can trigger via `/restart-mixer` → telnet `shutdown`).
- **Liquidsoap → Controller / UI**:
  - `now-playing.json` — written from `music_meta.on_metadata(on_meta)`. Hook is on `music_meta` — the **pre-cross** handle captured before `music` is wrapped in `cross(...)`. Hooking the post-cross source fires twice per transition because the custom `dj_transition` passes `initial_metadata=` into both `fade.in` and `fade.out`, freezing the UI one song behind. `on_metadata` is used instead of `on_track` because `on_track` gets swallowed by source switches (request queue → auto playlist).
- **Controller → Web UI**: HTTP. The `useStationFeed` hook (`web/hooks/useStationFeed.js`) polls `/now-playing` and `/state` every 5s.
- **Controller state**: `session.json` — the live DJ session (a chat-history JSON, see `broadcast/session.js`) the controller rewrites as tracks play and the DJ talks; archived sessions land in `state/sessions/<id>.json` on roll. Controller-internal — not read by Liquidsoap.
- **Browsers → Icecast**: direct `<audio src="…/stream.mp3">`. The `useMediaSession` hook wires lock-screen / headphone / CarPlay controls to the player, with artwork served from the controller's `/cover/:id` proxy.

Anything that needs to flow between the controller and Liquidsoap must go through one of these files — there is no socket or RPC channel.

### Controller (`controller/src/`, ESM Node.js)

Source is grouped by domain. `server.js`, `config.js`, `settings.js`, and `context.js` sit at the root; everything else lives under `routes/`, `middleware/`, `music/`, `broadcast/`, `audio/`, `llm/`, and `skills/`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [perminder-klair/subwave](https://github.com/perminder-klair/subwave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
