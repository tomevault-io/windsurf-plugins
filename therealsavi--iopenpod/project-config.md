---
trigger: always_on
description: iOpenPod is an iPod Classic sync tool that reads/writes Apple's proprietary iTunesDB and ArtworkDB binary formats. The app uses a PyQt6 GUI to browse and manage iPod music libraries.
---

# iOpenPod Copilot Instructions

## Project Overview
iOpenPod is an iPod Classic sync tool that reads/writes Apple's proprietary iTunesDB and ArtworkDB binary formats. The app uses a PyQt6 GUI to browse and manage iPod music libraries.

## Architecture

### Binary Parsers (`iTunesDB_Parser/` and `ArtworkDB_Parser/`)
Both parsers follow the **same recursive chunk-based pattern**:
- `parser.py` - Entry point accepting file path or file-like object
- `chunk_parser.py` - Router using `match` statements to dispatch by 4-byte chunk type (e.g., `mhbd`, `mhit`, `mhod`)
- `mh*_parser.py` - Individual parsers for each chunk type using `struct.unpack("<I", ...)` for little-endian binary reading
- `constants.py` - Maps for chunk types, MHOD types, and iTunes version identifiers

**Key pattern**: Every parser returns `{"nextOffset": int, "result": dict}` to enable recursive child parsing. New chunk parsers must follow this convention.

### GUI (`GUI/`)
- `app.py` - Main window with `ThreadPoolSingleton` for background loading, `Worker`/`WorkerSignals` for async tasks
- `widgets/` - PyQt6 components: `MBGridView` (album grid), `MBListView` (track list), `sidebar` (navigation)
- `imgMaker.py` - Decodes RGB565 artwork from `.ithmb` files using NumPy/Pillow

### Data Flow
1. Binary `.itunesdb`/`.artworkdb` files → Parsers → JSON (`idb.json`, `artdb.json`)
2. JSON → GUI loads via `AlbumLoaderThread`/`TrackLoaderThread` workers
3. Artwork: `mhiiLink` in track data references `imgId` in ArtworkDB → `.ithmb` file offset

### SQLite Database Writer (`SQLiteDB_Writer/`)
iPod Nano 6G/7G ignore binary iTunesCDB and read from SQLite databases in
`/iPod_Control/iTunes/iTunes Library.itlp/`.

- `sqlite_writer.py` — Orchestrates writing all 5 databases + cbk checksum
- `library_writer.py` — Library.itdb (tracks, albums, artists, composers, playlists, genres, avformat_info)
- `locations_writer.py` — Locations.itdb (file path mappings: item_pid → Fxx/filename)
- `dynamic_writer.py` — Dynamic.itdb (play counts, ratings, bookmarks)
- `extras_writer.py` — Extras.itdb (lyrics, chapters)
- `genius_writer.py` — Genius.itdb (empty tables)
- `cbk_writer.py` — Locations.itdb.cbk (HASHAB-signed SHA1 block checksums)

Activated when `DeviceCapabilities.uses_sqlite_db` is True (Nano 6G/7G only).
Timestamps use Core Data epoch (seconds since 2001-01-01, adjusted for timezone).

## iPod Database Format Reference

### File Locations on iPod
- `/iPod_Control/iTunes/iTunesDB` - Primary music database (never written by iPod firmware)
- `/iPod_Control/iTunes/Play Counts` - Play counts since last sync (iPod creates/updates this)
- `/iPod_Control/Artwork/ArtworkDB` - Album artwork metadata
- `/iPod_Control/Artwork/F*_1.ithmb` - Actual artwork images (RGB565 format)

### iTunesDB Chunk Hierarchy
```
mhbd (Database) → mhsd (DataSet, type 1-10) → 
  type 1: mhlt (TrackList) → mhit (Track) → mhod (strings)
  type 2: mhlp (PlaylistList) → mhyp (Playlist) → mhip (PlaylistItem)
  type 3: mhlp (PodcastList)
  type 4: mhla (AlbumList) → mhia (AlbumItem) → mhod
  type 5: mhlp (SmartPlaylistList)
  type 6: mhlt (empty stub, 0 children)
  type 8: mhli (ArtistList) → mhii (ArtistItem) → mhod type 300
  type 9: Genius CUID (raw string, no sub-chunks)
  type 10: mhlt (empty stub, 0 children)
```

### Critical mhit (Track Item) Fields - Currently Unimplemented
| Offset | Field | Size | Notes |
|--------|-------|------|-------|
| 20 | visible | 4 | 1=visible, other=hidden |
| 24 | filetype | 4 | "MP3 ", "M4A ", "M4P " as ASCII |
| 31 | rating | 1 | stars × 20 (0-100) |
| 36 | size | 4 | file size in bytes |
| 40 | length | 4 | duration in milliseconds |
| 44-48 | track_number/total_tracks | 4+4 | |
| 52 | year | 4 | |
| 56 | bitrate | 4 | e.g., 128, 320 |
| 60 | sample_rate | 4 | value × 0x10000 |
| 64 | volume | 4 | -255 to +255 adjustment |
| 80 | play_count | 4 | (iPod doesn't update this directly) |
| 84 | play_count_2 | 4 | plays since last sync |
| 88 | last_played | 4 | Mac timestamp |
| 92-96 | disc_number/total_discs | 4+4 | |
| 104 | date_added | 4 | Mac timestamp |
| 156 | skip_count | 4 | |
| 160 | last_skipped | 4 | Mac timestamp |
| 208 | media_type | 4 | 1=audio, 2=video, 0x20=music video |
| 248 | gapless_data | 4 | gapless playback data |
| 256 | gapless_track_flag | 2 | |
| 258 | gapless_album_flag | 2 | |
| 288 | album_id | 4 | links to mhia (u32 per libgpod) |
| 352 | mhii_link | 4 | links to ArtworkDB mhii entry |
| 480 (0x1E0) | artist_id | 4 | links to mhii in artist list (MHSD type 8) |
| 500 (0x1F4) | composer_id | 4 | per-track composer ID |
| 360 (0x168) | unk_flag | 4 | always 1 in all tested databases (libgpod writes 1) |

### Discovered mhip (Playlist Item) Fields
| Offset | Field | Size | Notes |
|--------|-------|------|-------|
| 0x0C | child_count | 4 | |
| 0x10 | podcast_group_flag | 2 | 0x000=normal, 0x100=podcast group |
| 0x14 | group_id | 4 | unique MHIP identifier |
| 0x18 | track_id | 4 | references mhit track_id |
| 0x1C | timestamp | 4 | Mac timestamp |
| 0x20 | group_id_ref | 4 | references another MHIP's group_id |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheRealSavi/iOpenPod](https://github.com/TheRealSavi/iOpenPod) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
