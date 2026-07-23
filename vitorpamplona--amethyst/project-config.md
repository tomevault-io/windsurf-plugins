---
trigger: always_on
description: Amethyst is a Nostr Client for Android that was made for Android-only and has been slowly switching
---

# Amethyst

## Project Overview

Amethyst is a Nostr Client for Android that was made for Android-only and has been slowly switching
over to a Kotlin Multiplatform project. The main modules are: `quartz`, `commons`, `amethyst`,
`desktopApp`, `cli`, plus the audio-rooms transport stack `quic` + `nestsClient`. Quartz should
contain implementations of Nostr specifications and utilities to help implement them. Commons stores
shared code between Amethyst Android (`amethyst`) and Amethyst Desktop (`desktopApp`). The Desktop
App is designed to be mouse first and so uses a completely different screen and navigation
architecture while sharing the back end components with the android counterpart. `cli` ships `amy`,
a non-interactive JVM command-line client that drives the same `quartz` + `commons` code — used by
humans, agents, and interop tests. `quic` is a from-scratch pure-Kotlin QUIC v1 + HTTP/3 +
WebTransport client (no JNI, no BouncyCastle), built because no Android-compatible Java QUIC library
exists. `geode` is a standalone JVM Nostr relay (Ktor) built on quartz's
relay-server code; smaller modules are `benchmark` (Android macrobenchmarks),
`relayBench` (head-to-head relay benchmark — boots geode, strfry and other
relay binaries, replays a shared deterministic corpus, measures ingest/query/
NIP-77 sync; `./relayBench/run.sh`, see `relayBench/README.md`) and
`quic-interop` (QUIC interop runner, lives at `quic/interop`). `nestsClient` runs
the audio-room protocol on top of `:quic` for the NIP-53 audio-rooms feature. It implements both IETF `draft-ietf-moq-transport-17` (under
`moq/`) and **moq-lite Lite-03** (kixelated's variant, under `moq/lite/`); the
production listener AND speaker paths both run on moq-lite to interop with the
nostrnests reference relay. The IETF code is kept as a reference + unit-test
implementation for any future IETF target; see
`nestsClient/plans/2026-04-26-moq-lite-gap.md`.

Canonical NIP specs live at <https://github.com/nostr-protocol/nips> — use
`/nip <number>` to pull a specific one (it fetches the spec file directly).

## Verify, Don't Guess

Don't assert a diagnosis you haven't reproduced. This repo gives you cheap
verification tools: `./gradlew test`, per-module test suites, and the `amy`
CLI (built partly to drive `quartz`/`commons` for interop checks). If a
claim is checkable in under a minute, check it before stating it — write
the failing case first, watch it fail, then explain.

## Architecture

```
amethyst/
├── quartz/         # Nostr KMP library (protocol only, no UI)
│   └── src/
│       ├── commonMain/    # Shared Nostr protocol, data models
│       ├── androidMain/   # Android-specific (crypto, storage)
│       ├── jvmMain/       # Desktop JVM-specific
│       └── iosMain/       # iOS-specific
├── commons/        # Shared UI components (convert to KMP)
│   └── src/
│       ├── commonMain/    # Shared composables, icons, state
│       ├── androidMain/   # Android-specific UI utilities
│       └── jvmMain/       # Desktop-specific UI utilities
├── quic/           # Pure-Kotlin QUIC v1 + HTTP/3 + WebTransport (audio-rooms transport)
│   └── src/
│       ├── commonMain/    # Protocol, frame/packet codecs, TLS state machine
│       ├── jvmAndroid/    # JCA-backed AEAD + UDP socket actuals
│       └── commonTest/    # RFC vector + adversarial tests
├── nestsClient/    # Audio-room client (production runs on moq-lite; IETF MoQ kept as reference)
│   └── src/
│       ├── commonMain/    # MoQ session, NestsListener, audio glue
│       └── jvmAndroid/    # Opus encode/decode, AudioRecord/AudioTrack
├── geode/          # Standalone JVM Nostr relay (Ktor) on quartz's relay-server code
├── desktopApp/     # Desktop JVM application (layouts, navigation)
├── amethyst/       # Android app (layouts, navigation)
└── cli/            # Amy — non-interactive CLI (JVM only, no Compose)
```

**Sharing Philosophy:**
- `quartz/` = Nostr business logic, protocol, data (no UI)
- `commons/` = Shared code for every front end (Android, Desktop, iOS, and the
  headless `cli`): domain models, state holders, ViewModels, the relay client,
  shared services, **and** the Compose UI that ≥1 GUI front end renders. The
  package taxonomy, the CLI-safe / UI boundary, and a "where does my code go?"
  guide are documented in **`commons/ARCHITECTURE.md`** — read it before adding
  a new package or dropping code into `commons`.
- `quic/` = Transport library (QUIC + HTTP/3 + WebTransport); reusable for any
  KMP project that needs MoQ. Has no Android-framework dependencies.
- `nestsClient/` = MoQ + audio-rooms client; takes `:quic` as transport,
  Quartz for crypto, `MediaCodec` / `AudioRecord` / `AudioTrack` for audio.
- `amethyst/` & `desktopApp/` = Platform-native layouts and navigation
- `cli/` = Thin assembly layer over `quartz/` + `commons/` (no new logic
  allowed). May also depend on `:geode` (for `amy serve`, which embeds the
  standalone relay); never on `:amethyst` or `:desktopApp`.

**Plans per module:** design docs for new subsystems live in the owning
module's `plans/YYYY-MM-DD-<slug>.md` (e.g. `cli/plans/`, `commons/plans/`).
The global `docs/plans/` folder is frozen — don't add new plans there.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vitorpamplona/amethyst](https://github.com/vitorpamplona/amethyst) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
