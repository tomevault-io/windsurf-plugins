---
trigger: always_on
description: Navidrome plugin (WASM) that reads embedded star-rating tags from MP3, FLAC, Ogg-Vorbis, Opus, WAV, DSF, M4A/AAC and WMA files and writes them to Navidrome via the Subsonic `setRating` API. Navidrome doesn't import embedded ratings on its own; this plugin bridges file tags and the Navidrome user-rating system.
---

# nd-rating-sync — project context

Navidrome plugin (WASM) that reads embedded star-rating tags from MP3, FLAC, Ogg-Vorbis, Opus, WAV, DSF, M4A/AAC and WMA files and writes them to Navidrome via the Subsonic `setRating` API. Navidrome doesn't import embedded ratings on its own; this plugin bridges file tags and the Navidrome user-rating system.

## File layout

| File | Responsibility |
|------|---------------|
| `main.go` | Entry points — lifecycle init (`OnInit` clears the in-progress guard on load), scheduler callback registration, and `runSyncStep` / `runSyncStepUntil` (one budgeted slice of a sync; checks/refreshes the `sweep-active` overlap guard, reschedules a continuation when the budget is hit) via `ratingPlugin` |
| `config.go` | Config types (`pluginConfig`, `libraryConfig`, `userConfig`) and `loadConfig()` |
| `cursor.go` | `syncCursor` (resumable position carried in the scheduler payload), `parseCursor` / `marshal`, `callBudget` (10 s — see *Sync execution* for why this is much lower than Navidrome's hardcoded 30 s limit), and `deadlineCheckEvery` (1 — songs between deadline checks; raising it lets slow songs blow past both the 10 s budget and the host 30 s wall) |
| `scanner.go` | Sync engine — `runSyncChunk` (walks library/user pairs until the deadline; loads each pair's threshold and applies the **LastScanAt gate** — skips a fresh pair whose library has not been rescanned since the stored threshold, without saving), `processPairChunk` (one pair from a song offset, takes the threshold as a param, checks the deadline after each song), and `processSong`. `extractStarsFromFile` returns a `fileReadResult` (`tagFound` / `tagAbsent` / `fileUnreadable`) so I/O failures, unsupported extensions, and parser panics never trigger `clear_rating_if_untagged`. Files are read via `readAudioMetadata`, which dispatches to a per-format extractor that touches only the metadata-bearing region of the file (`maxMetadataReadBytes` = 16 MiB per format, vs the audio body which can be many GiB). `dispatchParser` recovers panics from any container parser so one hostile file can't kill the whole sync. |
| `paths.go` | Filesystem-mount resolution and file matching — `resolveMountPoint` (config `libraryId` → `host.LibraryGetLibrary(int32).MountPoint`), `buildFileIndex` (recursive `os.ReadDir` walk keyed by `sizeKey(size, ext)`), `matchFile` (song → file by exact size+suffix; ambiguous/missing → not found). See **File access** below. |
| `state.go` | KV-backed state — `loadLastSynced` / `saveLastSynced` (incremental threshold; KV key `"last-synced:" + url.QueryEscape(libraryID) + ":" + url.QueryEscape(username)` so a `:` in either component can't collide with a different tuple), plus the overlap-guard helpers `sweepInProgress` / `markSweepActive` / `clearSweepActive` (`sweep-active` heartbeat key, `sweepStaleAfter` = 2 min). All KV failures fail open. |
| `subsonic.go` | Subsonic API domain — response types (incl. `subsonicSong.Size`, used to locate the file under the mount), `fetchSongPage`, `setRating` |
| `library.go` | `libraryLastScan` — wraps `host.LibraryGetLibrary` (empty libraryID → `LibraryGetAllLibraries`, newest `LastScanAt`) for the change-detection gate; returns `(zero,false)` on any uncertainty so the gate fails open. `cachedLibraryLastScan` memoises it per `runSyncChunk` call. |
| `id3.go` | ID3v2 tag parsing (`parseID3v2Rating`, `id3v2SyncsafeSize`) and the partial-read extractor `extractID3v2Metadata` — reads the 10-byte header, takes the syncsafe tag size, reads exactly that many bytes and stops. Also exposes `readID3v2TagAt` for WAV/DSF delegation. |
| `flac.go` | FLAC + Vorbis comment parsing (`parseFLACVorbisComments`, `parseFLACRating`, `ratingFromVorbisComments`) plus the partial-read extractor `extractFLACMetadata` — walks 4-byte metadata block headers and `Seek`s past non-VORBIS_COMMENT bodies (PICTURE blocks can be many MiB) so we never load the audio frames. Comment count is clamped to `maxVorbisComments` (1024) to bound allocations. |
| `ogg.go` | Ogg page walker (`extractOggPackets`) and Vorbis/Opus comment dispatch (`parseOggVorbisRating`) plus the partial-read extractor `extractOggMetadata` — reads the first `oggMetadataReadHint` bytes (512 KiB) which by spec covers the comment packet wherever it lands in the leading pages. |
| `wav.go` | WAV RIFF chunk walker (`parseWAVRating`) plus the partial-read extractor `extractWAVMetadata` — walks chunk headers and `Seek`s past audio (`data`/`fmt `) chunks; synthesises a minimal RIFF/WAVE container holding only the `id3 ` chunk. Chunk-size arithmetic stays in `uint64` before narrowing to `int` so a high-bit-set `uint32` can't sign-wrap on 32-bit `wasip1`. |
| `dsf.go` | DSD Stream File parser (`parseDSFRating`) plus the partial-read extractor `extractDSFMetadata` — reads the 28-byte DSD header, follows the embedded ID3 offset (which can sit GiB into the file past all the DSD samples) and reads only the tag. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [e1025735/nd-rating-sync](https://github.com/e1025735/nd-rating-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
