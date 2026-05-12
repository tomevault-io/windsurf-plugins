---
trigger: always_on
description: Telegram voice memo transcription bot with a Decision Atlas web dashboard. Users send voice memos or files via Telegram, the bot transcribes with AssemblyAI (speaker diarization, auto language detection), and optionally syncs to S3. Text messages are saved as notes. The React + Bun frontend renders the Decision Atlas (markmap overview, D3 tree views, competitor analysis, executive report).
---

# CLAUDE.md

## Project Overview

Telegram voice memo transcription bot with a Decision Atlas web dashboard. Users send voice memos or files via Telegram, the bot transcribes with AssemblyAI (speaker diarization, auto language detection), and optionally syncs to S3. Text messages are saved as notes. The React + Bun frontend renders the Decision Atlas (markmap overview, D3 tree views, competitor analysis, executive report).

## Key Commands

```bash
# Run bot + dashboard (port 8080)
uv run server/telegram_bot.py

# Manual transcription (standalone CLI, stays in root)
source .venv/bin/activate
python transcribe.py -i recording.m4a
python transcribe.py -f /path/to/recordings/
python transcribe.py -i recording.m4a --force-overwrite

# Drive watcher (standalone daemon, stays in root)
python drive_watcher.py              # daemon mode
python drive_watcher.py --dry-run    # list unprocessed files

# Frontend dev
cd web && bun install && bun run dev  # dashboard on :5173, proxies /api to :8080

# Frontend build
cd web && bun run build               # outputs to web/dist/

# Docker
docker compose up --build             # bot + dashboard on :8080

# Atlas data sync (requires INSFORGE_API_KEY)
bun scripts/sync-atlas-to-db.ts           # sync git-dirty files (auto-snapshots first)
bun scripts/sync-atlas-to-db.ts --all     # force sync all files
bun scripts/sync-atlas-to-db.ts market moat  # sync specific keys

# Atlas snapshots (requires INSFORGE_API_KEY)
bun scripts/snapshot-atlas.ts                          # create snapshot for __default__
bun scripts/snapshot-atlas.ts --label "before rewrite" # create with custom label
bun scripts/snapshot-atlas.ts --list                   # list recent snapshots
bun scripts/snapshot-atlas.ts --restore <id>           # restore from snapshot
bun scripts/snapshot-atlas.ts --prune --keep 20        # delete old, keep latest 20

# Server setup (Linux)
chmod +x setup_server.sh && ./setup_server.sh

# Company OS agent deployment (Ubuntu)
chmod +x deploy.sh && ./deploy.sh

# S3 sync (runs via cron every 5 min, or manually)
scripts/sync-all.sh              # run all syncs
scripts/sync-transcripts.sh      # notesly-transcripts → peakmojo-company-os
scripts/sync-pull.sh             # S3 → local (by-dates, context, skills)
scripts/sync-push-brain.sh       # local brain → S3

# Automated cron jobs (all times PDT, team is PST/PDT)
# Every 5 min: sync S3 + process new files, notify ONLY if new files found
scripts/auto-process.sh
# Daily 8AM PDT: morning briefing (schedule, prep notes, action items)
scripts/daily-prep.sh
# Every 3h (11AM,2PM,5PM,8PM PDT): action items digest (skip if unchanged)
scripts/action-items.sh

# Start Claude Code with Telegram channel (from ~/company-os/peakmojo)
claude --channels plugin:telegram@claude-plugins-official --dangerously-skip-permissions
```

## Architecture

### Server (`server/`)
- `server/telegram_bot.py` — Main entry point: Telegram bot + FastAPI on port 8080
- `server/api_server.py` — FastAPI app (`/api/status`, `/api/health`, `/api/atlas/*`, SPA serving)
- `server/bot_state.py` — Shared state singleton (module status, counters, errors)
- `server/src/transcription/transcriber.py` — AssemblyAI integration, speaker diarization, transcript caching
- `server/src/models/transcription.py` — TranscriptionSegment data class

### Web (`web/`) — React + Vite + Bun
- `web/src/App.tsx` — Main app with view state machine (overview, d3, competitor, executive-report)
- `web/src/hooks/useAtlasData.ts` — Data fetching hook for atlas dimensions + JSON data
- `web/src/components/MarkmapView.tsx` — Markmap mindmap overview
- `web/src/components/D3TreeView.tsx` — D3 tree renderer for individual dimensions
- `web/src/components/CompetitorView.tsx` — Competitor evolution stage view
- `web/src/components/ExecutiveReport.tsx` — 13-slide executive report deck
- `web/src/components/Sidebar.tsx` — Navigation sidebar
- `web/src/components/TopBar.tsx` — Title bar with level buttons
- `web/vite.config.ts` — Proxy `/api` to `:8080` in dev mode

### Atlas Data (`data/reports/data/`)
- `dimensions.json` — Metadata for all 8 decision dimensions
- `market.json`, `product.json`, etc. — Tree data for each dimension
- `competitor.json` — Competitive landscape evolution stages

### Scripts (`scripts/`)
- `scripts/sync-all.sh` — Run all sync jobs in sequence
- `scripts/sync-transcripts.sh` — Copy new transcripts from `s3://notesly-transcripts/by-dates/` → `s3://peakmojo-company-os/peakmojo/by-dates/`
- `scripts/sync-pull.sh` — Pull by-dates + context from S3 → local (`~/company-os/peakmojo/`)
- `scripts/sync-push-brain.sh` — Push local brain files → S3
- `scripts/auto-process.sh` — Cron every 5 min: S3 sync + detect/process new files, Telegram notify only when new files found
- `scripts/daily-prep.sh` — Cron daily 8AM PDT: morning briefing with schedule, prep notes, action items per team member

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [baryhuang/company-os](https://github.com/baryhuang/company-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
