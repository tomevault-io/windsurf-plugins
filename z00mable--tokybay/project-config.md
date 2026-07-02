---
trigger: always_on
description: C# .NET 10 console app for scraping and converting audiobooks (M4B/MP3) from multiple websites.
---

# TokyBay

C# .NET 10 console app for scraping and converting audiobooks (M4B/MP3) from multiple websites.

## Stack

- **.NET 10**, C# with nullable reference types and primary constructors
- **Spectre.Console** — all console output (markup with `[green]...[/]` etc., status spinners, tables, figlet)
- **Xabe.FFmpeg** — audio conversion and segment merging; binaries via `Xabe.FFmpeg.Downloader`
- **Newtonsoft.Json** — JSON parsing of API responses
- **Microsoft.Extensions.DependencyInjection** — DI container
- **Microsoft.Extensions.Configuration** — configuration via `appsettings.json` with binder
- **Microsoft.Extensions.Http** — `IHttpClientFactory` for typed HttpClients

## Architecture

### Strategy Pattern (Scraper)

```
IScraperStrategy                          (Scraper/Abstractions/)
    └── BaseScraperStrategy               (Scraper/Base/)
            ├── TokybookStrategy          (Scraper/Strategies/) — tokybook.com
            ├── DropboxTracksStrategy     (Scraper/Strategies/) — sites with JS `tracks = [{ chapter_link_dropbox }]` structure:
            │                             zaudiobooks.com, freeaudiobooks.top
            ├── AudioSourceTagStrategy    (Scraper/Strategies/) — sites with `<source type="audio/mpeg">` or `<a href="*.mp3">` structure:
            │                             goldenaudiobook.net, fulllengthaudiobooks.net, bigaudiobooks.net,
            │                             findaudiobook.com, bookaudiobook.net, hotaudiobooks.com, audiozaic.com,
            │                             appaudiobooks.com
            ├── PlaylistAudiobookStrategy (Scraper/Strategies/) — all sites with `data-playlist` JSON attribute:
            │                             hdaudiobooks.net
            └── AudioAzStrategy           (Scraper/Strategies/) — Next.js site with tracks JSON in streaming data:
                                          audioaz.com
```

- `ScraperFactory` selects the matching strategy via `CanHandle(url)`
- `ScraperConfig` controls parallelism parameters (defaults: 3 parallel downloads, 2 conversions, 5 segments/track)
- Runtime overrides in `Program.cs`: MaxParallelDownloads=5, MaxParallelConversions=3, MaxSegmentsPerTrack=8

### Download Pipeline

Downloads and conversions run decoupled via `Channel<T>` + `SemaphoreSlim`:
1. Download tasks write completed tracks into a bounded channel
2. Conversion tasks read from the channel and invoke FFmpeg
3. Channel is closed after `Task.WhenAll(downloadTasks)`

### Track Types

- `SegmentedTrackData` — for HLS streams (`.m3u8` → `.ts` segments → merge via FFmpeg concat)
- `DirectFileTrackData` — for direct MP3/audio downloads (zaudiobooks, goldenaudiobook); when the source is already in the target format, a copy-conversion runs to embed metadata without re-encoding

### Metadata Pipeline

`BaseScraperStrategy` provides shared metadata infrastructure used by all strategies.

For all non-Tokybook strategies, metadata is collected in two stages before downloading:

**Stage 1 — MP3 tag enrichment** (`EnrichFromFirstTrackTagsAsync`):
- Runs `ffprobe` on the first chapter URL to read existing ID3 tags without downloading the file
- Maps: `artist` → `Author`, `date` → `Year`, `comment` → `Description` (skips chapter references like "Chapter 1")
- Only fills empty fields — never overwrites

**Stage 2 — HTML extraction** (`ExtractCommonMetadata`):
- Only fills fields still empty after Stage 1
- `og:image` → `CoverArtUrl`
- `og:description` → `Description`
- `<script type="application/ld+json">` with `@type:"Audiobook"` → all fields (AudioAZ)
- `ld+json` `headline` field → author via `ExtractAuthorFromHeadline()` (WordPress/Yoast sites)
- `<link rel="preload" as="image">` → `CoverArtUrl` fallback (fulllengthaudiobooks, appaudiobooks)
- H1 title → author as last resort

Other shared helpers:
- `DownloadCoverArtAsync(url, folder)` — downloads cover once to `_cover.{ext}`, returns temp path
- `BuildMetadataParams(bookMetadata, trackData, hasCoverArt)` — returns FFmpeg `-metadata` flags (title, album, artist, album_artist, track, genre, comment, publisher, date, cover art)
- Cover art is passed to FFmpeg as a second input (`-map 0:a -map 1:v -c:v copy -disposition:v attached_pic`)
- Cover art temp file is always cleaned up via `finally` after the conversion pipeline completes

**Tokybook** gets richer metadata directly from the `post-details` API response (`authors`, `narrators`, `coverImage`, `description`, `publisher`) — no HTML scraping or ffprobe needed.

### Data Model

```
AudiobookMetadata (abstract)
│   Title, FolderPath
│   Author, Narrator, CoverArtUrl, Description, Publisher, Year   ← populated by ffprobe tags, HTML, or API response
├── SimpleAudiobookMetadata       — ChapterUrls: List<string>
└── StreamingAudiobookMetadata    — Tracks: List<TrackInfo>, StreamToken, AudioBookId

TrackData (abstract)
│   TrackTitle, SanitizedTitle, TrackNumber, TotalTracks
├── SegmentedTrackData            — TempFolder, FolderPath, TsSegments: List<string>
└── DirectFileTrackData           — FilePath, FolderPath

TrackInfo                         — Src, TrackTitle
UserSettings                      — DownloadPath, FFmpegDirectory, ConvertToMp3, ConvertToM4b
```

### Services


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [z00mable/TokyBay](https://github.com/z00mable/TokyBay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
