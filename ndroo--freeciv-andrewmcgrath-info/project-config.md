---
trigger: always_on
description: A Freeciv 3.2.4 longturn server running on Fly.io (`freeciv-longturn`) with 23-hour turns, email notifications, and a status page.
---

# Freeciv Longturn Server

## Project Overview
A Freeciv 3.2.4 longturn server running on Fly.io (`freeciv-longturn`) with 23-hour turns, email notifications, and a status page.

## Deployment
- **Platform**: Fly.io, app name `freeciv-longturn`
- **Deploy**: `fly deploy` (builds Docker image and restarts)
- **SSH**: `fly ssh console --app freeciv-longturn -C "command"`
- **Restart**: `fly apps restart freeciv-longturn`
- **Volume**: `/data/saves` persists across deploys (saves, sqlite db, logs)
- Quoting through `fly ssh` is painful — use base64 encoding for complex scripts

## Key Files
- `start.sh` — Main startup script, handles resume, FIFO, auto-take, turn notifications
- `longturn.serv` — Game settings (23hr timeout, player list)
- `turn_notify.sh` — Sends HTML email to all players on turn change
- `turn_reminder.sh` — Nudge emails 2hrs before deadline to players who haven't clicked Turn Done
- `crontab` — Cron schedule installed to `/etc/crontabs/freeciv`, runs `generate_status_json.sh` every 5 minutes
- `entrypoint.sh` — Starts crond as root, then execs `start.sh` as freeciv
- `generate_status_json.sh` — Extracts game data from save files into `status.json` and per-turn `api/{turn}.json`
- `www/index.html` — Client-side status page, renders from `status.json`
- `manage_players.sh` — Player management (DB auth, welcome emails, mid-game add)
- `fix_turn_timer.sh` — Manual override to set turn end time (e.g. `./fix_turn_timer.sh 4` for 4am)
- `change_gold.sh` — Change a player's gold via lua command
- `email_enabled.settings` — Set to `true` or `false` to enable/disable all emails

## Server Architecture
- FIFO at `/tmp/server-input` for sending commands to freeciv-server
- Background processes: auto-take on connect, turn change watcher, auto-saver (5min), turn reminder, HTTP server (busybox httpd on 8080)
- Status page runs via busybox crond every 5 minutes; turn changes also trigger it directly
- Healthcheck at `/cgi-bin/health` — returns 503 if status.json is >7 minutes old (for external monitoring like Pingdom)
- `turn_start_epoch` file tracks when the current turn's timeout started counting

## Save File Format
- Gzipped text files at `/data/saves/lt-game-{turn}.sav.gz`
- `save-latest.sav.gz` is used for resume on restart
- Key fields in save: `phase_seconds` (seconds elapsed in current turn), `turn`, `year`
- Timeout stored as `"timeout",value,default,"status"` format
- Map is 70x140 (xsize=70, ysize=140), wraps both axes

### Save File Structure
The save is a gzipped plaintext file with INI-style sections. Key sections:
- `[map]` — Terrain rows as `t0000="..."` through `t0139="..."` (first occurrence per row is terrain)
- `[playerN]` — Player data (nation, gold, government, diplomacy, etc.)
- After each player section: `nunits=N`, `u={header}`, then N comma-separated unit rows
- Unit row format: `id,x,y,"facing",nationality,veteran,hp,homecity,"type_by_name",...`

### Terrain Characters
` `=Ocean, `:`=Deep Ocean, `g`=Grassland, `p`=Plains, `f`=Forest, `h`=Hills,
`m`=Mountains, `d`=Desert, `j`=Jungle, `s`=Swamp, `t`=Tundra, `a`=Arctic,
`+`=Lake. Walkable land: `g p f h d s j t`

### Editing Save Files (Add Players, Change Attributes)
The most reliable way to modify game state is editing the save file directly.
FIFO lua commands get garbled for anything longer than ~200 chars, and server
commands like `playernation` and `set` are blocked mid-game. Save file edits
always work.

**Process:**
1. Force a save to capture current state:
   ```
   fly ssh console --app freeciv-longturn -C "sh -c 'echo save > /tmp/server-input; sleep 3'"
   ```
2. Download the save locally:
   ```
   fly ssh console --app freeciv-longturn -C "sh -c 'cat /data/saves/save-latest.sav.gz'" > /tmp/save.sav.gz
   gzip -dc /tmp/save.sav.gz > /tmp/save.txt
   ```
3. Edit the plaintext file (see examples below)
4. Upload and restart:
   ```
   gzip -c /tmp/save.txt > /tmp/save-edited.sav.gz
   cat /tmp/save-edited.sav.gz | base64 | fly ssh console --app freeciv-longturn \
     -C "sh -c 'base64 -d > /data/saves/lt-game-N.sav.gz && cp /data/saves/lt-game-N.sav.gz /data/saves/save-latest.sav.gz'"
   fly apps restart freeciv-longturn
   ```

**Adding a new player mid-game:**
1. Create DB auth first: `./manage_players.sh create <user> <pass> <email>`
2. In the save file, find the last `[playerN]` section
3. Add a new `[playerN+1]` section (copy structure from an existing player)
4. Set: `name`, `username`, `nation`, `gold`, `is_alive=TRUE`, `style_by_name`
5. Set `nunits=N` and add unit rows for starting units (Settlers, Warriors, Explorer, Diplomat)
6. Place units on separate walkable tiles, 5+ tiles from all other players
7. Use the terrain grid to find a good spot (look for clusters of `g`, `p`, `f`, `h`)
8. Update files for future deploys: `longturn.serv`, `start.sh` (aitoggle list), `manage_players.sh` (PLAYERS array)

**Common edits:**
- Change nation: `nation="British"` (also update `style_by_name` to match)
- Change gold: `gold=50`
- Add units: increment `nunits`, add rows before the `}` closing the unit table
- Change alive status: `is_alive=TRUE` or `is_alive=FALSE`

**Nation → Style mapping (common):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ndroo/freeciv.andrewmcgrath.info](https://github.com/ndroo/freeciv.andrewmcgrath.info) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
