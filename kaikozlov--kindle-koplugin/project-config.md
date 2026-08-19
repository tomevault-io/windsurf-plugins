---
trigger: always_on
description: **kindle.koplugin** is a [KOReader](https://koreader.rocks/) plugin that lets Kindle device owners browse and read their Kindle-native library books inside KOReader. It does this by:
---

# AGENTS.md — kindle.koplugin Project Instructions

## Project Overview

**kindle.koplugin** is a [KOReader](https://koreader.rocks/) plugin that lets Kindle device owners browse and read their Kindle-native library books inside KOReader. It does this by:

1. **Scanning** the Kindle's on-device document library (KFX files in `/mnt/us/documents/`)
2. **Decrypting** DRM-protected books using on-device key extraction
3. **Converting** KFX → EPUB via a bundled ARM CPython runtime and Python helper
4. **Caching** converted EPUBs for fast re-opening
5. **Presenting** a native `BookList` Kindle Library launched from one synthetic file-browser entry

KOReader itself is the architectural source of truth for UI/lifecycle behavior. `REFERENCE/kobo.koplugin/` may provide ideas, but do not copy its virtualization shims when current KOReader has a native extension point.

---

## Architecture

KOReader only sees **real document paths**. `KINDLE_VIRTUAL://` is legacy migration data, never a live document/file-browser path.

```
┌─────────────────────────────────────────────────────┐
│ KOReader (Lua)                                       │
│ main.lua                                             │
│   FileManager plugin init                            │
│     └─ minimal FileChooser hook → Kindle Library/    │
│                            │                         │
│                            ▼                         │
│                    native BookList                   │
│                            │                         │
│                select book / explicit open           │
│                            ▼                         │
│ filemanagerutil.openFile(real source/cache path)     │
│     └─ narrow resolver refreshes derived EPUB only   │
│        when that known Kindle path is stale/missing  │
│                            │                         │
│                            ▼                         │
│   ├─ KOReader chooses DocumentRegistry provider      │
│   ├─ KOReader owns DocSettings/History/Collections   │
│   └─ Reader lifecycle events                         │
│       ├─ DocSettingsLoad → Kindle → KOReader pull    │
│       └─ CloseDocument captures identity;            │
│          final SaveSettings → KOReader → Kindle push │
└──────────────────────┬──────────────────────────────┘
                       │ JSON CLI
┌──────────────────────▼──────────────────────────────┐
│ kindle-helper (C launcher + bundled ARM CPython)     │
│ python/kindle_helper.py                              │
│ KFX conversion / DRM / exact position translation    │
└──────────────────────┬──────────────────────────────┘
                       │ DRM init / native progress
┌──────────────────────▼──────────────────────────────┐
│ Kindle firmware services                             │
│ KFXVoucherExtractor + crypto hook / ReaderSDK agent  │
└─────────────────────────────────────────────────────┘
```

### Data Flow

```
User opens Kindle Library
  → FileChooser synthetic folder launches KindleLibrary BookList
  → browsing uses cc.db/scan metadata only (NO conversion or DRM side effects)
User selects a book
  → virtual_library model resolves the Kindle entry
  → filemanagerutil.openFile(real source/cache path)
  → open_file_ext refreshes/prepares only a known stale/missing Kindle-derived EPUB
  → one-time legacy sidecar migration runs if an old virtual sidecar exists
  → KOReader chooses the native provider (MuPDF/CREngine/etc.)
  → KOReader loads native DocSettings
  → onDocSettingsLoad may update position before ReadSettings
  → ReaderRolling/Paging restores position normally
  → CloseDocument captures the mapped Kindle book/path
  → ReaderRolling writes final position during SaveSettings
  → plugin onSaveSettings pushes that final state to Kindle
```

**Do not patch** `lfs.attributes`, `ffiUtil.realpath`, `DocumentRegistry`, `ReaderUI:showReader`, `ReaderUI:onClose`, or `DocSettings` to emulate files. If a feature appears to require that, first re-read current `REFERENCE/koreader/` for a native lifecycle/UI seam.

## Tech Stack

| Component | Language | Notes |
|-----------|----------|-------|
| KOReader plugin frontend | Lua | Runs inside KOReader's LuaJIT environment |
| KFX→EPUB conversion | Python | kfxlib from Calibre KFX Input plugin, run by bundled ARM CPython |
| DRMION decryption | Python | DeDRM ion.py + pycryptodome |
| DRM key extraction orchestration | Python | Shells out to device JVM with LD_PRELOAD hook |
| DRM voucher extraction | Java (tiny) | ~30 lines, runs on device's `cvm` JVM |
| AES key interception | C (tiny) | ~60 lines, LD_PRELOAD hook, pre-compiled as static asset |
| KOReader integration | Lua | Native `BookList` + `DocSettingsLoad` + close-capture/final-`SaveSettings`; narrow reversible FileChooser discovery + real-path open resolver hooks |

---

## Directory Layout

```
/
├── AGENTS.md
├── README.md
├── _meta.lua
├── main.lua                       ← plugin lifecycle + menus + sync events
├── KOREADER_TEST_COMMIT           ← pinned KOReader Lua contract for tests
├── python_build.sh
│
├── lua/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kaikozlov/kindle.koplugin](https://github.com/kaikozlov/kindle.koplugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
