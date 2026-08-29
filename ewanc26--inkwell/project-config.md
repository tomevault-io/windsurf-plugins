---
trigger: always_on
description: Guidance for agents working on Inkwell, a native reader and writer for the Standard.site publishing ecosystem on AT Protocol. This monorepo contains all three clients: iOS (`iOS/`), Android (`Android/`), and the marketing/legal site (`website/`).
---

# AGENTS.md

Guidance for agents working on Inkwell, a native reader and writer for the Standard.site publishing ecosystem on AT Protocol. This monorepo contains all three clients: iOS (`iOS/`), Android (`Android/`), and the marketing/legal site (`website/`).

## Principles

1. **KMP-first architecture** — all code that can be shared between platforms MUST be written in Kotlin in the `shared/` KMP module. iOS consumes shared code via thin Swift wrappers in `SharedKMP.swift` over the `InkwellShared.xcframework`. Only code that is fundamentally unable to be ported — platform UI (SwiftUI views, Compose composables), platform security storage (Keychain, EncryptedSharedPreferences), and platform-specific system integrations (OAuth browser flows, background task scheduling, notification managers) — stays native to its platform. Before writing any business logic, data transformation, format conversion, markdown parsing, facet handling, content model, or network orchestration in Swift, ask whether it can go in shared KMP instead. If it can, it must.
2. **Platform fidelity** — each platform's native UI conventions, accessibility, and UX come first. Don't port iOS UI patterns to Android or vice versa without explicit adaptation. The KMP boundary is logic, not presentation.
3. **Protocol truth over README claims** — if the README says something the code doesn't actually do, the code is right and the README is wrong. Update both together.
4. **Honest stubs** — unimplemented features say so explicitly in the UI, return errors, or are gated behind runtime checks. Never silent no-ops or fabricated successes.
5. **Security material stays put** — OAuth tokens, DPoP keys, PKCE state, and session secrets belong in platform secure storage (Keychain / EncryptedSharedPreferences) only. Never in UserDefaults, SharedPreferences, logs, or git.
6. **AT Protocol correctness** — cross-reference the atproto spec and upstream reference implementations for wire formats (DAG-CBOR, MST, XRPC, DID documents). Don't infer protocol behavior from observation.
7. **No duplication** — if a piece of logic already exists in the codebase, reuse it. Two independent implementations of the same rule silently drift apart. This applies doubly across the KMP boundary: never reimplement shared KMP logic in platform-native code.
8. **Modular file structure** — every file owns one clear responsibility. Each feature lives in its own folder containing its ViewModel, View, and helpers as separate files. Reusable UI components (toolbars, pickers, rendering views) get their own files. Keep files under ~400 lines; split before exceeding. The former offenders (`LoginStateManager.swift`, `ContentProvider.swift`, `PdsRepository.kt`, `PostDetailScreen.kt`) were split along responsibility lines in 2.0 and are no longer exempt — hold new code to the same limit.

## Current state

- **iOS** (`uk.ewancroft.Inkwell`): primary SwiftUI client, marketing version `2.5.0` build `58`. OAuth with DPoP complete. Reader, Discover, Writer tabs functional. Leaflet blocks, Markpub/Offprint/pckt rendering. Markdown parsing/serialization, facet conversion, verification URL building, link scanning, and constellation deduplication delegated to shared KMP via `SharedKMP.swift`. Background notification polling. Comments, subscriptions, recommends, publication/document verification, blob handling, and native report tooling implemented. Writer: split-pane editor with live markdown preview, formatting toolbar, image upload, loss reporting, and document editing. AltStore distribution with live screenshots.
- **Android** (`uk.ewancroft.inkwell`): Kotlin/Compose client, version `2.5.0` versionCode `12`. OAuth complete. Reader, Discover, Writer functional. Reader publication theming (Leaflet rich theme, legacy palette, basicTheme cascade). Leaflet blocks with rich-text facets. Markpub markdown rendering with headings, lists, code blocks, blockquotes, images, task lists, horizontal rules, and inline formatting. pckt/Offprint block arrays converted to markdown with facet-aware inline formatting via shared KMP. Bluesky post embeds with live fetching and author/image/link/quote rendering. Standard.site post embeds with document fetch and cover image. Comments, subscriptions, recommends, and interactions (likes/reposts/replies) implemented. WorkManager background notification polling. Verification URL/link-scan logic delegated to shared KMP; Constellation pagination/deduplication delegated to shared KMP. Writer: formatting toolbar, live preview toggle, loss reporting banner, and document editing. F-Droid self-hosted repo with fastlane screenshots. Native report tooling implemented.
- **Website** (`inkwell.ewancroft.uk`): SvelteKit/Vercel marketing, legal, and OAuth-metadata site. Hosts live `/client-metadata.json`, AltStore `source.json`, F-Droid repo index, and web-optimized screenshots for both platforms.

## Roadmap

Public kanban board: https://github.com/users/ewanc26/projects/3

Use the kanban board to track work across columns: **Backlog** → **Todo** →

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ewanc26/inkwell](https://github.com/ewanc26/inkwell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
