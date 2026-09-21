---
trigger: always_on
description: Router for coding agents. Knit is an offline Android **mesh messenger** (Kotlin/Compose) that runs
---

# AGENTS.md — Knit

Router for coding agents. Knit is an offline Android **mesh messenger** (Kotlin/Compose) that runs
**Wi-Fi Aware (NAN) + Bluetooth LE** simultaneously behind one `MeshTransport` seam
(`CompositeMeshTransport`), no Google Nearby / GMS. This file *points* to context — load a `.agents/`
file only when its trigger matches. Full design detail lives in `docs/`.

## Identity

You are a senior Android/Kotlin engineer on a deliberately bleeding-edge toolchain (AGP 9.3.0 /
Kotlin 2.4.0, Koin DI). Favor correctness, wire/convergence safety, and matching the surrounding style
over cleverness. Start with `.agents/context/architecture.md` for the subsystem map and data flow.

## Context routing

- **Before any build / dependency / tooling change:** READ `.agents/context/toolchain.md` — the
  bleeding-edge choices (Koin-not-Hilt, the Kotlin-2.4 override, detekt/ktlint/Kover/Room as Gradle
  plugins) are deliberate; don't "fix" them.
- **Before / after running Gradle:** obey `.agents/rules/build-and-test.md` (which task when, JDK 21,
  lockfile regen). Command list: `.agents/context/commands.md`.
- **Before touching `app/src/main/baseline-prof.txt`, the `:baselineprofile` module, or the
  `nonMinifiedRelease` build type:** READ `.agents/context/baseline-profile.md` — the generator is quarantined
  behind `-Pknit.baselineProfile=true` on purpose, so that the release build consumes a committed text file
  and stays byte-reproducible for F-Droid. Don't wire the plugin into `:app`.
- **Before touching release signing, `packaging`/`ndk` config, `.gitattributes`, or anything else that
  changes release-APK bytes:** READ `.agents/context/distribution.md` — Play and F-Droid ship different
  artifacts under different keys, and F-Droid byte-compares its own rebuild against ours, so the release
  build must not depend on the build machine (no NDK on the APK path, no foojay JDK download, no Git LFS).
- **When editing any Kotlin/Compose/data code:** obey `.agents/rules/coding.md`.
- **When adding to `CHANGELOG.md`'s `## Unreleased`, or writing a fastlane changelog:** obey
  `.agents/rules/changelog.md` — two sentences, about forty words, run through the `humanizer` skill.
  A `PreToolUse` hook blocks the edit otherwise. Shipped sections are a record; never restyle them.
- **When touching `mesh/`, `protocol/`, or `data/`:** obey `.agents/rules/mesh.md`, then READ the
  relevant reference — `.agents/context/mesh-transport.md` (radios / NAN / BLE),
  `.agents/context/wire-format.md` (CBOR wire), `.agents/context/store-and-forward.md` (custody /
  convergence), `.agents/context/e2e-encryption.md` (crypto). If a change can only be made by *breaking*
  the wire, don't — park it in `docs/NEXT_WIRE_BREAK.md` (the staging list, so a future break carries them
  all at once) and find the additive route per `docs/WIRE_COMPAT.md`.
- **When touching `mesh/bluetooth/BleSideChannel`, `SideCarousel`, `SideScanPolicy`, `SideCapableTracker`,
  `BleFastRoutePolicy`, `BleAdvertPayload`'s flags byte, or `BluetoothMeshTransport.fastFanout`/`fastSend`:**
  READ ADR 2026-09.sjaa and the "page carousel" section of `.agents/context/mesh-transport.md`. The BLE side
  channel carries `shouldFastFanout` frames on non-connectable extended-advertising pages — one
  `FastFrameCodec` unit per 236-B page, never a chain, never presence, never a DM-form frame — gated per
  peer on the advert's flags byte and dark in release behind `BuildConfig.BLE_SIDE_PLANE` until its device
  trial (CHECK `.agents/memory/roadmap.md`). The receive scan is Off for an all-linked clique with nothing
  streaming (ADR 2026-09.u8qj) — the link copy already reaches every linked peer; don't widen that gate. `hasFastPlane` is now true for Bluetooth: the link copy the
  composite used to send lives inside the transport's `fastFanout`/`fastSend`; don't add it back upstream.
- **When touching `mesh/lora/` or `mesh/bluetooth/meshtastic/` (the LoRa/Meshtastic bridge):** READ
  `.agents/context/lora-bridge.md` — a Meshtastic board over BLE GATT extends the **Nearby room and 1:1
  DMs** over LoRa as a fast-plane-only `MeshTransport` child, shipped visible since 2.5.0 behind
  `BuildConfig.LORA_PLANE` and off until the user pairs a board (ADR 038 + 039, introduced by ADR
  2026-09.6gtm — it is no longer a gate that keeps anything out of shipped builds). `mesh/lora/` is
  pure/JVM-tested; the only `android.bluetooth.*` importer is `mesh/bluetooth/meshtastic/MeshtasticGatt`.
  **Before touching `DmAutoReplyPolicy`, `Destination.Reply`, `OutboundFrame.to`, or `MeshtasticLink.send`'s
  `to`:** READ ADR 2026-09.4n5p — a Meshtastic DM to a set-up board is answered once with a fixed unicast
  (once per sender per day, once per 30 s for anybody, the room's air share, never from a stock or a
  dedicated-slot board), and that reply is the only unicast the plane sends.
- **When touching `linkpreview/`, `net/`, `mesh/protocol/LinkPreviewBlob`, or anything that opens an
  Internet socket outside the spool plane:** READ ADR 2026-09.n752 (and 2026-09.7x8k: a send holds up to 5 s
  for the card its link is fetching, or the share sheet never carries one; a LoRa thread takes a card exactly

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getknit/knit](https://github.com/getknit/knit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
