---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal music catalog app for browsing, rating, tagging, and randomly picking albums. The core feature is the "Surprise Me" random album picker with filters. Google Sheets serves as the persistent data store (read and write); H2 is used as a runtime cache/database. The music library data was scanned once from an external drive via a Python script — the scanner is a one-time prerequisite, not part of the running application. **The app is deployed and live** on Google Cloud Run, scaled to zero when idle, with Google Sheets as the persistent source of truth.

## Current State

All tasks (0–18) from `task-list.md` are complete. The app is deployed to Cloud Run.

**Done:**
- `music_scanner.py` — One-time Python scanner that produced `catalog.json` (prerequisite, already run)
- `catalog.json` — Scanned library (~176 artists, ~2830 albums, ~31K tracks across 7 genres) at project root
- `plan.md` — Full architecture and design document
- `task-list.md` — Sequential implementation tasks (Tasks 0-10)
- **Backend skeleton** — Spring Boot app with Gradle build, Flyway migration, H2 database, application.yml
- **Domain entities** — ArtistEntity, AlbumEntity, SongEntity, TagEntity with JPA mappings
- **Repositories** — ArtistRepository, AlbumRepository (with EntityGraph), SongRepository, TagRepository
- **Catalog import** — CatalogImportService (JSON → DB), CatalogAutoImporter (auto-imports on first startup if DB empty), POST /api/catalog/import endpoint
- **Catalog DTOs** — Java records in `dto.catalog` package: Catalog, Genre, Artist, Album, Stats, ImportResult
- **Artist CRUD API** — ArtistService, ArtistController (full CRUD + favorite toggle + tag management)
- **Album CRUD API** — AlbumService, AlbumController (full CRUD + grade + favorite + tags + rich filtering via AlbumFilterParams)
- **Tag CRUD API** — TagService, TagController (list, create, delete)
- **DTOs** — ArtistDto, ArtistCreateDto, ArtistUpdateDto, AlbumDto, AlbumSummaryDto, AlbumCreateDto, AlbumUpdateDto, AlbumFilterParams, GradeDto, SongDto, TagDto, TagCreateDto
- **Exception handling** — NotFoundException, NoMatchException, GlobalExceptionHandler with @ControllerAdvice (404, 400 validation, 500)
- **Browse API** — BrowseService, BrowseController (genres with counts, artists by genre, albums by artist, tag stats, favorites, full stats with grade distribution)
- **Random Album API** — RandomPickService, RandomController (single random album, multiple random albums, all album filters supported)
- **JPA Specifications** — AlbumSpecs extracted as reusable static utility class (artistGenreEquals, byArtist, hasAnyTag, gradeGte, isFavorite, isUnrated, etc.)
- **Browse DTOs** — BrowseGenreDto, BrowseTagDto, BrowseStatsDto, BrowseFavoritesDto

- **Frontend shell** — React 19 + TypeScript + Vite 7, TanStack Query v5, React Router 7, Tailwind CSS 4, Lucide icons; Vite proxy to :8080 in dev, builds to backend static resources
- **Frontend structure** — `src/types/index.ts` (all TS interfaces), `src/api/client.ts` (typed axios calls), `src/hooks/` (useArtists, useAlbums, useBrowse, useRandomAlbum), `src/components/Layout.tsx + Sidebar.tsx`, `src/pages/DashboardPage.tsx` (live stats)
- **Shared components** — StarRating (clickable 1-5 stars), FavoriteToggle (heart icon), TagBadge (pill with remove), FilterBar (genre/grade/favorite/tag filters), AlbumCard (summary card), ArtistCard (artist card)
- **Browse page** — Genre grid with inline accordion expansion showing artists, drill-down to artist detail
- **Artist pages** — ArtistListPage (filtered grid of ArtistCards), ArtistDetailPage (header + favorite + tags + album grid)
- **Album pages** — AlbumListPage (FilterBar + AlbumCard grid), AlbumDetailPage (header + star rating + favorite + tags + song table)

- **Random Pick page** — "Surprise Me" button with genre/grade/favorite filters, shows full album detail with inline rating/tagging, "Roll Again" flow
- **Favorites page** — Favorite artists and albums grids with empty state, inline unfavorite toggles
- **Tags page** — Tag cloud with sort (name/usage), create/delete tags, click tag to show associated artists and albums

**Deployment phase (Tasks 9-18), all done:**
- **Google Sheets sync** — `GoogleSheetsClient`/`SheetMapper` (chunked writes via `values.append` for rows beyond the first, since `values.update` never grows a sheet's grid past its current row count; 429 retry with backoff); write path pushes Artists+Albums synchronously after every mutating commit (Songs only on structural changes) via `SheetSyncListener`; read path restores the DB from Sheets on an empty boot, or seeds from `catalog.json` and pushes if the sheet is blank; `POST /api/catalog/sync/push`, `POST /api/catalog/sync/pull`, `GET /api/catalog/sync/status`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexshamrai/music-cat](https://github.com/alexshamrai/music-cat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
