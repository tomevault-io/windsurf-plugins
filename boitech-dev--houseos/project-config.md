---
trigger: always_on
description: You install, run, fix and change HouseOS for the person who owns this folder. Answer in their
---

# HouseOS: instructions for the AI agent working on this copy

You install, run, fix and change HouseOS for the person who owns this folder. Answer in their
language. Ask before installing system packages or services, using `sudo`, or touching their
TV, speakers, router or firewall. New to them? docs/START-HERE.md is a short introduction.

## Read first

1. **Installing:** follow **AGENT-INSTALL.md** (checks, questions, the welcome and tour messages,
   fixes). For more depth:
   - docs/START-HERE.md, then what they bring: AGENT-INSTALL.md §5.
   - Then the guide for their computer:
     - Linux with Docker: docs/DOCKER.md (`./houseos.sh`).
     - Windows: docs/WINDOWS.md (WSL 2, step by step with checks).
     - Linux without Docker: docs/SETUP.md.
   - Then docs/INTEGRATIONS.md, docs/DEVICES.md (their TVs, speakers and remotes, by brand), and .agent/STATE.md.
2. **Running it** (update, back up, find what's wrong): the next section.
3. **Changing code:**
   - docs/ARCHITECTURE.md (how the parts fit), then only the relevant section of
     docs/CODE-INDEX.md (every module, function and route, with line links).
   - docs/CHANGING-HOUSEOS.md: how a feature is built end to end, the checks, deploying and going
     back, and how changes Nox drafts are reviewed in Control Room → Changes.
   - The code and docs/ARCHITECTURE.md are the source of truth; docs/design/ has the product
     brief, the writing rules, the design system and each theme's direction.

## Running the house for them

Everything runs from this folder. Check with `./houseos.sh status` before and after.

| They ask | You do |
|---|---|
| "Update HouseOS" | `./houseos.sh update`. Music resumes at the same second; wait for a film to end. |
| "Back it up" | `./houseos.sh backup` → `backups/<date>/`. Private (keys, `.env`): suggest copying it to another disk. Restore: docs/DOCKER.md §9. |
| "Something's wrong" | `./houseos.sh status`, then `./houseos.sh logs [service…]`, then the table in docs/DOCKER.md → When something is off. *Control Room → Health* and *Logs* show the same from the app. |
| "A device doesn't work" | The section *When the user's device or setup doesn't work* below. |
| "The setup code" | `./houseos.sh setup-code` (only until the first account exists). |
| "Reach it from outside" | Tailscale (`tailscale serve --bg 8990`) or their reverse proxy: docs/DOCKER.md §3. Never a router port forward. |
| "Voice on the graphics card", "the torrent player", "buttons in Control Room" | `./houseos.sh gpu on`, `torrents on`, `buttons on` (each has `off`). |

- Settings, keys and accounts live in the app (*Control Room*), not in files: guide them there.
- `docker compose down` keeps everything; `down -v` deletes all data. Never run `-v` unless they
  ask for a fresh start, after a backup.
- Record what you did (never secret values) in .agent/STATE.md.

## What this is

A fresh, clean copy, with nothing from anyone else's house:
- no accounts or credentials;
- no configured stream add-on or debrid API token;
- no provider login, devices, history or hosting.

Everything the user needs to supply is listed in AGENT-INSTALL.md §5. Never ask the user to
paste secrets into chat. They enter keys in HouseOS's Control Room, where they are stored
encrypted.

## How it is put together (one minute)

- **Services** (Docker, one container per service):
  - `api` is FastAPI plus the built React UI.
  - `worker` runs music jobs.
  - `fetch` resolves YouTube, SoundCloud and radio, with no database and no keys.
  - `audio` is the mpv speaker bridge.
  - `maintenance` runs reminders, song genres and the weekly film index.
  - `cinema-worker` checks film versions in a sandbox; `cinema-observer` follows what the TV plays.
  - `tusd` takes uploads in pieces.
  - `relay` serves media to TVs.
  - `voice` is faster-whisper.
  - `codex` and `claude` are sign-in bridges.
  - `https` is the built-in door on :8443.
  - `db` is MariaDB.
  - `helper` (only after `./houseos.sh buttons on`) runs `houseos.sh`'s fixed actions for the Control Room.
- **Talking to each other:**
  - Services talk through the database (durable jobs, leases, confirmations) and small Unix
    sockets in the state volume.
  - The UI gets live updates from an event stream.
- **Backend:** `backend/houseos/`, one module per area.
  - Music: `music.py`, `audio.py`, `music_genre.py`.
  - Cinema: `cinema.py`, `cinema_explore.py`, `film_index.py`; Watch → Web (any video link): `cinema_web.py`, `fetcher_web.py`.
  - Home: `household.py`.
  - Assistant: `assistant*.py`.
  - Smart home: `home.py`.
- **Frontend:** `frontend/src/`, one file per room with its CSS beside it (`music.tsx`,
  `watch.tsx`, `household.tsx`, `me.tsx`, `control.tsx`…; the list is in docs/FRONTEND.md).
  - Product and writing rules: docs/design/BRIEF.md, docs/design/COPY.md.
  - **The design system** (`frontend/src/design/`, docs/design/SYSTEM.md): every control,
    pattern, overlay and state comes from `./design` (`Button`, `Field`, `Sheet`,
    `ConfirmSheet`, `Form`, `List`/`ListRow`, `State`, `Notice`, `Problem`…). A room's own CSS
    only lays things out, in `@layer rooms`, with tokens. `npm run build` fails on a raw

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [boitech-dev/HouseOS](https://github.com/boitech-dev/HouseOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
