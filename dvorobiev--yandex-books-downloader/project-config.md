---
trigger: always_on
description: A Chrome Extension (Manifest V3) that downloads books from [Bookmate](https://bookmate.com) as
---

# GitHub Copilot Instructions — Bookmate Downloader Extension

## Project Overview

A Chrome Extension (Manifest V3) that downloads books from [Bookmate](https://bookmate.com) as
EPUB files or audio tracks. Everything runs entirely in the browser — no native binaries, no external libraries,
no build step. The `src/` folder is loaded directly as an unpacked extension.

## File Roles

| File | Role |
|---|---|
| `src/manifest.json` | MV3 manifest — permissions, content-script declaration, service-worker registration (`"type": "module"` enables ES module imports in the service worker) |
| `src/background.js` | Service worker entry point — `chrome.runtime.onConnect` message handler; dispatches to `downloadBook`, `downloadSerial`, or `downloadAudiobook` based on `msg.bookType` (`BookType` enum); contains `sendBackZipInChunks` which streams finished ZIP bytes back to the popup in 1 MB chunks via `zip-chunk`/`zip-end` messages |
| `src/lib/booktype.js` | `BookType` frozen enum — `BOOK`, `SERIAL`, `AUDIO`, `COMICBOOK`, `SERIES`; imported by `background.js` and the `lib/` modules. **Also inlined verbatim in `content.js` and `popup.js`** (plain scripts cannot use ES module imports) — keep all three copies in sync |
| `src/lib/crc32.js` | CRC-32 lookup-table implementation (`CRC_TABLE`, `crc32`) |
| `src/lib/http.js` | Shared HTTP/file utilities — `READER_BASE` constant, `fetchWithCookie`, `blobToDataUrl`, `safeName`; imported by `bookmate.js` and `audiobook.js` to eliminate duplication |
| `src/lib/epub.js` | DEFLATE via `CompressionStream` + ZIP/EPUB binary assembler (`deflateRaw`, `assembleParts`, `buildEpub`, `buildZip` and helpers); imports `crc32`. `assembleParts` is a shared sync core used by both `buildEpub` and `buildZip` |
| `src/lib/decrypt.js` | AES-CBC decryption via Web Crypto API (`base64ToBytes`, `decryptValue`) and `extractClientParams` HTML parser |
| `src/lib/bookmate.js` | Bookmate download orchestration (`downloadBook`, `downloadSerial`); private helpers: `decryptMeta`, `parseOpfHrefs`, `downloadContentFiles`, `stripCssFiles`, `buildEpubFilename`, `saveEpubBlob`; imports from `epub.js`, `decrypt.js`, `merge.js`, `http.js` |
| `src/lib/merge.js` | In-memory EPUB episode merger (`mergeEpisodes`); regex-based XML manipulation of OPF manifest/spine and NCX navMap — mirrors Python `EpubMerger` |
| `src/lib/audiobook.js` | Audiobook download logic (`fetchAudiobookMeta`, `downloadAudiobook`); private helpers: `resolveTrack` (resolves track number + `.m4a` URL), `trackBaseName` (builds filename stem), `downloadAudiobookIndividual` (one `chrome.downloads` call per track), `downloadAudiobookAsZip` (fetches all tracks, builds ZIP, calls `onZipReady` callback); imports from `epub.js` and `http.js` |
| `src/content.js` | Injected into every `*.bookmate.com` page (plain script) — detects book pages, injects a download `<button>` into `.buttons-row__container`, handles SPA navigation via `window.navigation` |
| `src/popup.html` | Toolbar popup UI — dark theme, CSS variables, progress bar, scrollable log box |
| `src/popup.js` | Popup logic (plain script) — reads active-tab URL to extract book ID and `BookType`, persists `stripCss` via `chrome.storage.sync`, communicates with background via a long-lived port |

## Architecture & Data Flow

```
User action (popup button OR injected page button)
  │
  ▼
popup.js / content.js
  └─ chrome.runtime.connect({ name: 'bookmate-download' })
       └─ postMessage({ action: 'download', bookid, bookType, stripCss, maxBitRate })
             │              OR for audiobooks, first:
             │        postMessage({ action: 'audiobook-meta', bookid })
             │              └─ response: { type: 'audiobook-meta', trackCount, title }
             │              └─ popup shows choice if trackCount > 1, then sends download msg
             │
             ▼
         background.js  (service worker)
           1. Verify `bms` session cookie (chrome.cookies)
           2. Dispatch: BookType.AUDIO → downloadAudiobook(), BookType.SERIAL → downloadSerial(), BookType.BOOK → downloadBook()

  ── Regular book (downloadBook) ─────────────────────────────────────────
           3. GET reader.bookmate.com/<bookid>  → extract window.CLIENT_PARAMS.secret
           4. Parallel fetch: encrypted metadata + book info
           5. AES-CBC decrypt metadata fields that are arrays (container, opf, ncx)
           6. Regex-parse OPF manifest → list of content file hrefs
           7. Sequential GET of each content file (OEBPS/ base URL)
           8. Optionally zero-out .css files (stripCss flag)
           9. buildEpub() → valid ZIP/EPUB blob
          10. chrome.downloads.download() via data-URL
          11. port.postMessage({ type: 'success'|'progress'|'error', ... })

  ── Serial book (downloadSerial) ────────────────────────────────────────
           3. GET reader.bookmate.com/<bookid>  → extract window.CLIENT_PARAMS.secret
           4. Parallel fetch: book info + episodes list (/p/api/v5/books/<bookid>/episodes)
           5. For each episode (episode.uuid):
              a. GET /p/api/v5/books/<episode.uuid>/metadata/v4  → encrypted metadata
              b. AES-CBC decrypt (same serial secret for all episodes)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dvorobiev/yandex-books-downloader](https://github.com/dvorobiev/yandex-books-downloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
