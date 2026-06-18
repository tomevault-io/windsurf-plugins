---
trigger: always_on
description: > Coverage docs at `coverage/{STATE}/{sport}.md` are the living per-combo truth — see [Coverage System](#coverage-system) section.
---

# Teams United League Service — CLAUDE.md

> Coverage docs at `coverage/{STATE}/{sport}.md` are the living per-combo truth — see [Coverage System](#coverage-system) section.

## Overview

Multi-platform youth sports standings collection service for [Teams United](https://teams-united.com).
Collects league standings from 9 different scoring platforms, stores them in Firestore, and serves
them via Cloud Functions API + GCS-hosted dashboard.

**GCP Project:** `teams-united` (us-central1)
**Deploy VM:** `35.209.45.82:8080` — see [Deploy VM Access](#deploy-vm-access) below
**Firestore:** 3 main collections: `leagues`, `divisions`, `standings`
**Dashboard:** GCS-hosted HTML at `https://storage.googleapis.com/tu-league-dashboard/`

## Architecture

```
Cloud Scheduler (daily) → collectAll → iterates active leagues → adapter.collectStandings()
                                                                      ↓
User/API → collectLeague(leagueId) ─────────────────────────→ adapter.collectStandings()
                                                                      ↓
                                                              Firestore (divisions, standings)
                                                                      ↓
                                                              updateSheet → Google Sheets
                                                                      ↓
Cloud Scheduler (weekly) → seasonMonitor → health checks, season discovery, auto-dormant
```

### Cloud Functions (Gen2, Node 20)

| Function | Trigger | Purpose | Memory |
|---|---|---|---|
| `collectLeague` | HTTP POST `{leagueId}` | Collect one league | 1024MB* |
| `collectAll` | Cloud Scheduler (daily) | Collect all active leagues | 1024MB* |
| `getLeagues` | HTTP GET | List leagues with filters | 256MB |
| `getDivisions` | HTTP GET `?league=` | List divisions for a league | 256MB |
| `getStandings` | HTTP GET `?division=` | Get standings for a division | 256MB |
| `seasonMonitor` | Cloud Scheduler (weekly) | Detect stale/dormant leagues, discover new seasons | 256MB |
| `updateSheet` | POST (after collectAll) | Sync Firestore → Google Sheets | 256MB |
| `discoverGC` | HTTP POST | Discover GameChanger leagues via DuckDuckGo + API | 256MB |
| `discoverGroups` | HTTP POST | Discover GotSport division groups | 256MB |

\* collectLeague/collectAll need 1024MB for Puppeteer-based adapters (SC, GC). Deploy script: `scripts/deploy-memory-upgrade.sh`

### 9 Platform Adapters (`adapters/`)

| Adapter | Method | Sports | Config Keys |
|---|---|---|---|
| **gamechanger** | Browser (Puppeteer) | Baseball, Softball | `orgId`, `allOrgIds` |
| **sportsconnect** | Browser (Puppeteer) | Baseball (Little League, PONY) | `baseUrl`, `standingsTabId`, `programs[]` |
| **sportsaffinity** | JSON API | Soccer | `organizationId`, `seasonGuid` |
| **sportsaffinity-asp** | Browser (Puppeteer) | Soccer | `organizationId` (GUID), auto-discovers flights |
| **gotsport** | HTML scraping | Soccer | `leagueEventId`, `groups[]` |
| **tgs** | Browser/API | Soccer (ECNL, GA) | `eventId` |
| **demosphere** | HTML scraping | Soccer | `baseUrl`, `divisions[]` |
| **pointstreak** | HTML scraping | Baseball, Hockey | `leagueId`, `seasonId` |
| **leagueapps** | HTML scraping | Baseball, Soccer, Basketball, Lacrosse | `baseUrl`, `programs[]` |

### Key Files

- `index.js` — Cloud Function entry point (collectLeague, collectAll, getLeagues, getDivisions, getStandings)
- `registry.js` — Adapter registry
- `browser.js` — Shared Puppeteer launcher (v2 with frame-detached resilience)
- `season-monitor.js` — Weekly health checks + auto season discovery
- `sheets-sync.js` — Google Sheets sync
- `discover-gc.js` — GameChanger org discovery via DuckDuckGo search
- `discover-groups.js` — GotSport group auto-discovery
- `lib/age-group-parser.js` — Universal age group normalization (U4-U19, HS, Adult, etc.)
- `dashboard/` — Firebase-hosted ops dashboard

### Firestore Schema

**leagues** collection:
```
{
  name, sport, state, region,
  sourcePlatform, sourceConfig: { ... platform-specific ... },
  status: 'active' | 'dormant' | 'pending_config' | 'pending_tabid' | 'pending_groups' | 'deactivated_phase1' | 'template',
  autoUpdate, lastCollected, lastDataChange, lastStandingsHash,
  monitorStatus: 'healthy' | 'stale' | 'dormant' | 'error' | 'needs_attention',
  monitorNotes, lastMonitorCheck,
  seasonStart, seasonEnd, staleDays, discoveryConfig
}
```

**divisions** collection:
```
{ id, leagueId, seasonId, name, ageGroup, gender, level, platformDivisionId, status }
```

**standings** collection (keyed by `{divisionId}-{slugified-teamName}`):
```
{ teamName, position, gamesPlayed, wins, losses, ties, points, scored, allowed, differential, ... }
```

## Current State (April 27, 2026)

> **For real-time numbers, run a fresh four-bucket audit** via the `teamsunited-league-service-ops` Hermes skill — these are point-in-time snapshots that drift weekly.

### Stats (fresh as of 2026-04-27)
- ~5,800 leagues truly collecting (>0 divisions, recent `collectedAt`)
- ~17,354 `pending_adapter` (mostly SBL-universal placeholders awaiting the ScoreBookLive Firecrawl adapter)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [helpertubot/teams-united-league-service](https://github.com/helpertubot/teams-united-league-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
