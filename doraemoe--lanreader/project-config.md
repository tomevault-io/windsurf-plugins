---
trigger: always_on
description: Use this file as the source of truth for Codex-style work in this repository.
---

# LANreader Agent Guide

Use this file as the source of truth for Codex-style work in this repository.

## Start Here

- Prefer the repo scripts over retyping `xcodebuild` or CI commands:
  - `./scripts/bootstrap`
  - `./scripts/lint`
  - `./scripts/test-ios`
- The scripts use the normal Xcode folders by default.
- Set `LANREADER_USE_LOCAL_XCODE_ENV=1` if you explicitly want repo-local caches under `.codex/`.
- The current project, app, and scheme names are `LANreader`.

## Project Facts

- Default branch: `master`
- Xcode project: `LANreader.xcodeproj`
- Main app scheme: `LANreader`
- Secondary scheme: `Action`
- Test target: `LANreaderTests`
- CI selects Xcode 26.6 from `/Applications/Xcode_26.6.app`.
- CI simulator destination: `platform=iOS Simulator,OS=26.5,name=iPad Pro 11-inch (M5)`
- CI lint command: `swiftlint --strict`
- CI test command: `xcodebuild clean test -project LANreader.xcodeproj -scheme LANreader ... -skipMacroValidation`
- Current active GitHub workflows are `ci.yml` and `manual-ipa-release.yml`.
- Local unsigned IPA packaging is supported.
- Manual releases derive tags as `<MARKETING_VERSION>-<CURRENT_PROJECT_VERSION>`.

## Repo Map

- `LANreader/LANreaderApp.swift`: app entry point, logging bootstrap, app-wide tasks.
- `LANreader/Page/`: reader, paging, image display, archive details.
- `LANreader/Models/`: LANraragi response models and app-facing view models.
- `LANreader/Library/`, `LANreader/Category/`, `LANreader/Search/`, `LANreader/Setting/`: feature UIs.
- `LANreader/Service/`: LANraragi API client, translation, image handling, transaction observer.
- `LANreader/Database/`: GRDB database setup and records.
- `Action/`: action extension target.
- `LANreaderTests/`: XCTest coverage for services, reducers, reader positioning, and extracted feature logic.
- `ci_scripts/`: CI bootstrap helpers, including Swift macro trust config.

## Architecture Notes

- The app is a mixed SwiftUI/UIKit project, not a pure SwiftUI app.
- SwiftUI is used for feature composition and state-driven views, but performance-sensitive scrolling surfaces use UIKit wrappers.
- `UIArchiveList.swift`, `UIPageCollection.swift`, `UICacheView.swift`, and related cells/controllers use `UIViewControllerRepresentable`, `UIHostingController`, and `UICollectionView` to avoid the poor performance previously seen with pure SwiftUI `LazyHStack` and `LazyVStack` approaches.
- Treat the UIKit collection view layer as an intentional performance decision, not legacy code to rewrite away by default.
- The app is SwiftUI-based at the feature level and uses the Composable Architecture heavily across feature views.
- Dependencies are injected through `Dependencies` where the code already follows that pattern.
- Persistence uses `GRDB` and `GRDBQuery`.
- Networking and streaming logic live under `LANreader/Service/`.
- Reader rendering has special handling for animated images and HEIC conversion; inspect `ImageService.swift`, `UIPageCell.swift`, and `PageImageV2.swift` together before changing image behavior.
- Reader navigation crosses three ownership layers: `ArchiveReader.swift` owns feature state and intent, `ReaderPositioning.swift` owns pure index calculations, and `UIPageCollection.swift` performs UIKit scrolling. Inspect all three before changing page jumps, restore behavior, double-page layout, or RTL behavior.

## Reader and Cache Invariants

- LANraragi page numbers are one-based. Reader collection indices are zero-based and can diverge further when split-page items are inserted. Resolve server-facing page numbers through `PageFeature.State.pageNumber` or `sourcePageNumber` instead of treating them as collection indices.
- Preserve `ReaderNavigationSource` when requesting a jump. Initial restore and slider jumps are centered, while chapter, tap, keyboard, and automatic navigation are edge-aligned.
- Chapter metadata follows the full path from the LANraragi response to `ArchiveItem.toc`, then to `ArchiveCache.toc` for offline reading. Keep the property optional so older servers and existing cache rows remain valid.
- Opening a cached archive must remain offline. Do not add a network request to recover missing chapter metadata or other optional metadata; if it was not persisted when the archive was cached, leave the related UI unavailable.
- Persisted cache-model changes require an additive GRDB migration in `AppDatabase.swift` and backward-compatible decoding. Do not rewrite or rename an existing migration after it may have shipped.
- Tankoubon chapter navigation flattens the real TOCs from its contained archives. Offset each one-based local chapter page by the actual number of extracted pages from preceding archives, and ignore entries outside their source archive's extracted page range. Add the included archive title as a default chapter at its first page unless a valid manual chapter already starts there.

## Tankoubon Boundaries

- Treat IDs with the `TANK_` prefix as Tankoubons via `isTankoubonArchiveId`. Thumbnail, metadata, progress, update, and delete operations must use the `/api/tankoubons/...` endpoints instead of archive endpoints.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Doraemoe/LANreader](https://github.com/Doraemoe/LANreader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
