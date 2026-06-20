---
trigger: always_on
description: Analyze, clean up, and organize Apple Photos libraries. Find and report junk photos (screenshots, low-quality, burst leftovers, duplicates), analyze storage usage, generate photo timeline recaps, plan smart exports, analyze Live Photos, check iCloud sync, audit shared libraries, detect similar photos, curate seasonal highlights, and score face quality. All analysis operations are READ-ONLY on the database (safe). macOS only. Requires Python 3.9+ (stdlib only) and access to the Apple Photos SQLit
---


# Apple Photos Cleaner

Comprehensive toolkit for analyzing and cleaning up Apple Photos libraries. Goes beyond what Photos.app offers: intelligent junk detection, detailed storage analysis, duplicate finding with quality scoring, timeline recaps for storytelling, and smart export planning.

## Overview

Apple Photos is great at organizing and syncing photos, but it's not so great at cleanup. This skill fills that gap:

- **Library Analysis** — Get the big picture: counts, storage, date ranges, people, quality distribution
- **Junk Finder** — Identify screenshots, low-quality photos, burst leftovers, old screenshots
- **Duplicate Finder** — Find duplicates using Apple's detection + timestamp/dimension matching
- **Storage Analyzer** — Detailed breakdown by year, type, file format, growth trends, storage hogs
- **Timeline Recap** — Generate narrative summaries of photo activity for any date range
- **Smart Export** — Plan organized exports by year/month, person, album, or location; AppleScript export
- **Best Photos / Hidden Gems** — Surface high-quality photos you haven't favorited
- **People Analyzer** — Deep analysis of people: co-occurrence, trends over time, best photos per person
- **Location Mapper** — Cluster GPS coordinates into locations, identify trips, offline reverse geocoding
- **Scene Search** — Search by ML-detected content (beach, dog, food) or generate content inventory
- **Photo Habits** — Behavioral analytics: time-of-day, day-of-week, streaks, seasonal trends
- **On This Day** — See what you photographed on today's date in prior years
- **Album Auditor** — Find orphan photos, empty albums, overlap between albums
- **Cleanup Executor** — Batch move junk to trash via AppleScript with confirmation
- **Live Photo Analyzer** — Compare Live Photos vs stills, find conversion candidates, storage impact
- **Shared Library** — Analyze Shared Library vs personal: contributors, content splits, storage
- **iCloud Status** — Check iCloud sync coverage: synced vs local-only, large unsynced items
- **Similarity Finder** — Detect visually similar photos using computed quality feature vectors
- **Seasonal Highlights** — Curate the best photos per season using quality scores and favorites
- **Face Quality Scoring** — Rank face photos per person: find best/worst portraits

**Safety:** All operations are READ-ONLY database queries. No photos are modified or deleted without explicit user action.

## When to Use This Skill

Use when users mention:
- Cleaning up Photos / freeing up photo storage
- Finding duplicate photos
- Removing old screenshots
- Analyzing Photos library storage
- Finding junk or low-quality photos
- Organizing photo exports
- Getting photo timeline summaries ("what did I do last week?")
- Burst photo cleanup
- Finding storage hogs in Photos
- Finding best or hidden gem photos
- People in photos, who appears together
- Where photos were taken, travel, trips, locations
- Searching photos by content (beach, sunset, dog, food)
- Photo-taking habits, patterns, streaks
- "On this day" / photo memories from past years
- Album organization, orphan photos, album cleanup
- Actually deleting junk photos (batch cleanup)
- Live Photos analysis, converting Live Photos to stills
- Shared Library content, who contributed what
- iCloud sync status, unsynced photos, cloud coverage
- Finding similar-looking photos
- Seasonal photo highlights, best of each season
- Face/portrait quality, best/worst portraits per person

## Quick Start

All scripts work standalone. The Photos database is automatically located at:
`~/Pictures/Photos Library.photoslibrary/database/Photos.sqlite`

**Basic workflow:**
1. Run `library_analysis.py` to get overview
2. Run `junk_finder.py` to identify cleanup candidates
3. Run `duplicate_finder.py` to find duplicates
4. Use results to guide manual cleanup in Photos.app

## Commands

### 1. Library Analysis

Get comprehensive library statistics: counts, storage, date ranges, people, quality scores.

```bash
python3 scripts/library_analysis.py [--human] [--output FILE]
```

**Options:**
- `--human` — Human-readable summary instead of JSON
- `--output FILE` — Write JSON to file
- `--db-path PATH` — Custom database path
- `--library PATH` — Custom Photos library path

**Example Output:**
```
📊 APPLE PHOTOS LIBRARY ANALYSIS
==================================================

Total Assets: 12,453
Total Storage: 48.3 GB
Average Size: 4.1 MB
Date Range: 2020-01-15 to 2025-03-03

By Type:
  Photo: 11,234
  Video: 891
  Screenshots: 328
  Favorites: 456
  Bursts: 1,234

By Year:
  2025: 1,203 items, 5.2 GB
  2024: 3,456 items, 15.1 GB
  2023: 2,987 items, 12.4 GB
  ...

Top People:
  Jonah: 3,456 photos
  Silas: 3,234 photos
  ...
```

**Usage in Conversation:**

**User:** "How many photos do I have?"  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [and3rn3t/apple-photos-cleaner](https://github.com/and3rn3t/apple-photos-cleaner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
