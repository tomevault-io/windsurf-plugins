---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project

`wisp` is a SwiftUI iOS Nostr client. Bundle id `barrydeen.wisp`. Swift 5, iOS deployment target 26.4, supports iPhone/iPad/visionOS (`SUPPORTED_PLATFORMS = iphoneos iphonesimulator macosx xros xrsimulator`).

SwiftPM dependencies (resolved via Xcode, no `Package.swift` exists):
- `objectbox-swift-spm` — embedded event database
- `swift-secp256k1` (21-DOT-DEV) — Schnorr signing/verification + ECDH
- `breez-sdk-spark-swift` — Spark (self-custodial Lightning) wallet
- `giphy-ios-sdk` — GIF picker

## Build / run / test

This is an Xcode project — there is no `Package.swift`, no `make`, no CLI script. Open `wisp.xcodeproj` in Xcode, or use `xcodebuild`:

```
xcodebuild -project wisp.xcodeproj -scheme wisp -destination 'platform=iOS Simulator,name=iPhone 16' build
xcodebuild -project wisp.xcodeproj -scheme wisp -destination 'platform=iOS Simulator,name=iPhone 16' test
```

Tests under `wispTests/` use the Swift Testing framework (`import Testing`, `@Test`), not XCTest. Substantive coverage lives in `Nip44Tests`, `NSpamTests`, `RelaySettingsTests`, `SafetyTests`. UI tests under `wispUITests/` are XCTest.

## Source layout — important

The Xcode project mixes two file-management styles:

- `wisp/`, `wispTests/`, `wispUITests/` are `PBXFileSystemSynchronizedRootGroup`s — anything dropped into those folders on disk is automatically part of the target. The `wisp/` folder holds `wispApp.swift` (the `@main`), `ContentView.swift`, `Assets.xcassets`, `Resources/`, and most of the top-level UI screens (sidebar, compose FAB, group/DM views, thread view, splash, loading, live-stream views under `wisp/Live/`).
- **Most domain code (view models, repositories, NIP implementations, crypto) lives at the repo root, not under `wisp/`** (e.g. `FeedViewModel.swift`, `RelayPool.swift`, `NostrEvent.swift`, `EventStore.swift`, `Nip17.swift`, `Schnorr.swift`, …). These are added to the target via explicit `PBXFileReference` entries in `wisp.xcodeproj/project.pbxproj`. **When you add a new root-level Swift file, you must also add it to `project.pbxproj`** — it will not be picked up automatically. Files inside `wisp/` are exempt from this.

`EntityInfo-wisp.generated.swift` exists in two places: the repo root (compiled into the target) and `generated/` (output of the ObjectBox generator). When the entity model changes, the generator writes to `generated/` and `model-wisp.json` is updated; the root copy must be replaced to match.

## Bundled secrets (no xcconfig)

API keys ship as gitignored text files in `wisp/Resources/`:

- `wisp/Resources/breez-api-key.txt` (Breez Spark SDK)
- `wisp/Resources/giphy-api-key.txt` (Giphy)

Both have `.example` siblings checked in. `.gitignore` excludes the real files. `BreezConfig` and `GiphyConfig` read them from the bundle at startup, falling back to empty/hardcoded values. Do **not** introduce xcconfig + Info.plist injection for new secrets — follow the bundled-resource pattern.

`wisp/Resources/nspam/` ships the on-device LightGBM spam model (`model.txt`, `calibration.npz`) and is checked in. `wisp/Resources/bip39-english.txt` is the BIP-39 wordlist.

## App flow

App flow is driven by a single `@State` enum in `wisp/ContentView.swift`:

```
splash → (login via nsec/mnemonic) → onboarding → main
                                   ↘ (returning user, onboarding done) → loading → main
```

`MainView` is a five-tab `TabView` (home / wallet / search / messages / notifications). Each tab owns its own `NavigationStack`, and navigation pushes typed `Hashable` route values (`ProfileRoute`, `ThreadRoute`, `HashtagFeedRoute`, `NoteListFeedRoute`, `PeopleListFeedRoute`, `TrendingFeedRoute`, `LiveStreamRoute`) dispatched via `.navigationDestination(for:)`. A left-edge `SidebarDrawerView` overlays the active tab for account switching, settings sheets, and tab selection.

Account state is keyed by hex pubkey throughout. `NostrKey` stores the active keypair plus a per-pubkey list in the iOS Keychain (service `com.wisp.nostr`), and uses `UserDefaults` keys of the form `onboarding_done_<pubkey>`, `follow_pubkeys_<pubkey>`, `relay_scoreboard_v1_<pubkey>`, `latest_feed_ts_<pubkey>`, `profile_<pubkey>`, etc.

## Architecture

### Outbox-model relay routing (NIP-65)

This is the central architectural idea for the home feed. Rather than fanning every query out to every relay:

1. **Onboarding** (`OnboardingViewModel.startOutboxBuilding`): hit a hardcoded set of indexer relays (`indexer.nostrarchives.com`, `indexer.coracle.social`, `relay.damus.io`, `relay.primal.net`) to fetch the user's kind-0 (profile), kind-3 (contacts), then kind-10002 (relay lists) for every followed pubkey, in batches of 150.
2. **`RelayScoreBoard`** inverts that data into `relayAuthors: [relayURL: Set<authorPubkey>]`, taking up to `redundancy=3` write relays per author, then ranks relays by author count. Persisted to UserDefaults as a tab-delimited string list.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [barrydeen/wisp-ios](https://github.com/barrydeen/wisp-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
