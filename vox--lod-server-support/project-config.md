---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

LOD Server Support (LSS) — distributes LOD chunk data from servers to clients over a custom networking protocol. Supports both Fabric (client + server) and Paper (server only). Clients request distant chunks individually; the server reads them from disk or memory and streams serialized sections back, enabling LOD rendering mods to display terrain beyond vanilla render distance.

## Project Structure

Multi-project Gradle build with three subprojects:

```
lod-server-support/
├── common/   Pure Java utilities (no MC deps) — shared by fabric/ and paper/
├── fabric/   Fabric mod (client + server), Minecraft 26.1
└── paper/    Paper plugin (server only), Minecraft 1.21.11
```

## Build Commands

```bash
./gradlew :fabric:build -x runClientGameTest  # Build Fabric mod + Tier 1 & 2 tests
./gradlew :paper:shadowJar                    # Build Paper plugin JAR
./gradlew clean                               # Clean build artifacts
```

Output JARs:
- `fabric/build/libs/lod-server-support-fabric.jar` — Fabric mod (client + server)
- `paper/build/libs/lod-server-support-paper.jar` — Paper plugin (server only, shadow JAR)

## Test Commands

```bash
./gradlew :fabric:test -x runGameTest -x runClientGameTest  # Tier 1: JUnit unit tests only (fast, ~7s)
./gradlew :fabric:runGameTest                                # Tier 2: server gametests (starts dedicated server, ~13s)
./gradlew :fabric:test -x runClientGameTest                  # Tier 1 + 2 combined
./gradlew :fabric:runClientGameTest                          # Tier 3: client gametests (starts integrated server + client)
./gradlew :fabric:build -x runClientGameTest                 # Full build + Tier 1 + 2 tests
```

Tests only exist for the Fabric module. Paper is compile-only (manual testing required).

### Test Architecture

- **Tier 1** (`fabric/src/test/java/dev/vox/lss/`): JUnit 5 tests via `fabric-loader-junit`. Tests position packing, bandwidth limiter, config validation, column cache store, column timestamp cache, payload codecs, and negative/oversized payload edge cases.
- **Tier 2** (`fabric/src/gametest/java/dev/vox/lss/test/LSSGameTests.java`): 11 Fabric server gametests. Validates `RequestProcessingService` activation, diagnostics, command registration, and config loading inside a real dedicated server.
- **Tier 3** (`fabric/src/gametest/java/dev/vox/lss/test/LSSClientGameTests.java`): End-to-end client-server flow test. Validates handshake, session config, spiral scanning, chunk section receipt. Fabric Loom handles headless rendering automatically.

The system property `-Dlss.test.integratedServer=true` (set in fabric/build.gradle for gametest JVMs) allows `RequestProcessingService` to activate on integrated servers during testing.

## Architecture

### Entry Points

- **Fabric:** `LSSMod` — server initializer: registers payloads, starts `RequestProcessingService` on dedicated servers
- **Fabric:** `LSSClient` — client initializer: sets up client networking, commands, mod compatibility
- **Paper:** `LSSPaperPlugin` — plugin entry point: registers Plugin Messaging channels, starts `PaperRequestProcessingService`

### Networking Protocol (v15)

Batch model with 8 payload types. Fabric uses `LSSNetworking` with Fabric `StreamCodec`;
Paper uses `PaperPayloadHandler` with raw `FriendlyByteBuf` encoding. Both produce identical wire format.

**C2S:** `HandshakeC2SPayload` (capabilities bitmask) → `BatchChunkRequestC2SPayload` (batch of requestId + packed position + clientTimestamp tuples) → `CancelRequestC2SPayload` (cancel in-flight request) → `BandwidthUpdateC2SPayload` (client desired bandwidth)

**S2C:** `SessionConfigS2CPayload` (limits/config/rate limits/generation toggle) → `VoxelColumnS2CPayload` (requestId + MC-native sections + columnTimestamp) → `BatchResponseS2CPayload` (batch of responseType + requestId pairs — covers rate-limited, up-to-date, and not-generated) → `DirtyColumnsS2CPayload` (server-pushed change notifications)

Flow: client handshakes with capabilities bitmask (CAPABILITY_VOXEL_COLUMNS set if LSSApi has consumers) → server sends session config with rate limits → client scans in expanding spiral every second (20 ticks) and sends batched requests with clientTimestamp (-1 for unknown/first request, 0 for generation request, >0 for resync) → server routes by timestamp: sync-on-load path (clientTimestamp > 0 or -1) checks rate limiter then reads disk, generation path (clientTimestamp == 0) checks rate limiter then generates → server batches lightweight responses (rate-limited, up-to-date, not-generated) per tick; column data payloads are sent individually. After initial scan, server periodically pushes `DirtyColumnsS2CPayload` listing changed columns, client re-requests only those.

Chunk coordinates are packed as `((long)chunkX << 32) | (chunkZ & 0xFFFFFFFFL)`.

### Common Module (`common/`)

- `LSSConstants` — protocol version, channel IDs, wire format limits, status codes
- `LSSLogger` — SLF4J logging wrapper
- `SharedBandwidthLimiter` — fair per-tick bandwidth allocation across active players (global + per-player caps)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VoX/lod-server-support](https://github.com/VoX/lod-server-support) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
