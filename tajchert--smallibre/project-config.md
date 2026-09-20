---
trigger: always_on
description: Build a small native macOS ebook manager: quick local import, useful metadata, restrained EPUB personalization, and reliable reader transfers. Prefer focused Swift implementations and system frameworks over adding runtimes or general-purpose conversion stacks. macOS is the target; cross-platform support is not an accepted requirement. GPL-3.0-only. A local ignored `calibre/` checkout may exist as reference material; it is not a dependency or shipped implementation.
---

# Working on Smallibre

## Product direction

Build a small native macOS ebook manager: quick local import, useful metadata, restrained EPUB personalization, and reliable reader transfers. Prefer focused Swift implementations and system frameworks over adding runtimes or general-purpose conversion stacks. macOS is the target; cross-platform support is not an accepted requirement. GPL-3.0-only. A local ignored `calibre/` checkout may exist as reference material; it is not a dependency or shipped implementation.

Read the current code and README before using older design plans. Documents under `docs/superpowers/` include proposals and unimplemented stages. `docs/architecture/kindle-management.md` describes the first reader increment; `reader-reliability.md` supersedes its cache/helper/edit limitations.

## Architecture

| Location | Responsibility |
| --- | --- |
| `Sources/SmallibreApp` | SwiftUI interface with AppKit panels and WebKit preview. `AppModel` and `ReaderModel` are main-actor observable UI state. `Theme.swift` and `Components.swift` hold the design system: colour tokens, the cover artwork, shared rows, pills, form groups and button styles. Build screens from those instead of raw colours or ad-hoc chrome. |
| `Sources/SmallibreCore` | Book inspection, bounded ZIP handling, EPUB editing, SQLite library, metadata lookup, device operations and receipts. |
| `Sources/SmallibreReaderHelper` | Separate executable for potentially blocking mounted-device I/O; JSON request/response via `ReaderClient`. |
| `Sources/CSQLite`, `Sources/CZlib` | System library module maps; no downloaded package dependencies. |
| `Tests/SmallibreCoreTests` | Core regression tests and independently authored fixtures. |
| `Tests/SmallibreAppTests` | Library selection, editing, startup and reader gating regression tests. |
| `scripts/build-app.sh`, `Resources/Info.plist` | Release bundle assembly, helper inclusion, icon generation, identifiers and versions. |

`LibraryStore` is an actor owning library operations and its SQLite wrapper. Import stages a file, hashes it, detects/validates its format, deduplicates by SHA-256, saves its immutable original, and then inserts its database record. `EPUBEditor` prepares a separate archive; MOBI/AZW3 library exports keep original bytes. `MOBIMetadataEditor` only rewrites supported standalone device metadata, not book content.

Never put `fixedSize(horizontal: false, vertical: true)` on text inside a `NavigationSplitView` column: the column then reports an unbounded ideal height and every column is pushed out of the window. Verify interface changes by running the packaged app, not only by building it.

`ReaderModel` tracks connection state and serializes UI operations. `ReaderClient` starts the helper through `ReaderProcess`; `ReaderStore` performs device work. A process gate prevents overlapping helpers, with cancellation and deadlines. A kernel-blocked process can outlive a cancellation request: never interpret cancellation as proof a write did not happen.

## Invariants to preserve

- Keep imported originals immutable. Never overwrite an export destination. Back up and verify before device deletion or metadata replacement.
- Keep `BookOrganization` (tags, series/number, read state) library-only; organization edits must not change export bytes or conversion identity. Older records default to empty organization and unread.
- Bulk edits apply only explicitly chosen fields to visible selected books, rereading records and saving atomically. Keep single-book actions disabled for multiple selections and range anchors consistent across grid/list views.
- Use `LibraryQuery` for library search/filter semantics. Saved views persist query rules per library, not book IDs; device search remains separate.
- Identify books by bytes, not title. Revalidate file hashes, connection tokens and root identity before acting on cached device selections.
- Keep device I/O in the helper. Preserve path containment, symlink rejection and descriptor-relative mutation checks; do not simplify them to unchecked path operations.
- Preserve sidecars, annotations and device databases. Deletes are nonrecursive and apply only to selected files. KFX main-file backups are not complete package backups.
- Preserve operation receipts and uncertain outcomes. Do not automatically replay interrupted writes or remove uncertain staging files.
- Enable reader actions only after library initialization succeeds. `LibraryLocation` migrates legacy storage, preserves conflicting copies and repairs interrupted migration. Legacy names in migration and typography compatibility code are intentional.
- Treat ebook content as untrusted data. Keep resource limits, archive validation and XML protections. Current limits include 256 MB book/archive, 64 MB resource, 8 MB XML, and 10,000 ZIP entries; ZIP64 and DRM are unsupported.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tajchert/smallibre](https://github.com/tajchert/smallibre) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
