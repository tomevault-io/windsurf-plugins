---
trigger: always_on
description: - **Gradle multi-module project** exporting three libraries (`pumpx2-android`, `pumpx2-messages`, `pumpx2-shared`) plus utilities (`cliparser`) and a demo Android app (`sampleapp`).【F:settings.gradle†L19-L20】【F:build.gradle†L1-L35】
---

# PumpX2 repository orientation

## High-level layout
- **Gradle multi-module project** exporting three libraries (`pumpx2-android`, `pumpx2-messages`, `pumpx2-shared`) plus utilities (`cliparser`) and a demo Android app (`sampleapp`).【F:settings.gradle†L19-L20】【F:build.gradle†L1-L35】
- All modules share the `com.jwoglom.pumpx2` namespace. `shared` holds reusable utilities, `messages` implements the Tandem Bluetooth protocol, `androidLib` layers Android/BLE integration on top, and `cliparser`/`sampleapp` demonstrate usage.
- Build artifacts are configured for Maven publishing (local and GitHub Packages) and target Java 11 / Android API 26+.【F:build.gradle†L1-L35】【F:androidLib/build.gradle†L18-L65】

## Build & test quick-reference
- `./gradlew build` builds every module, producing AAR/JAR artifacts for distribution.【F:README.md†L63-L88】
- `./gradlew test` (or module-specific variants) runs the Java unit tests located in `messages/src/test` (JUnit 4).【F:messages/build.gradle†L20-L43】【83f740†L1-L10】
- `./gradlew publishToMavenLocal` publishes artifacts locally; publishing destinations are configured in `build.gradle`.
- `./gradlew cliparser:run --args '<command ...>'` executes the CLI parser for offline message parsing/encoding.【F:cliparser/src/main/java/com/jwoglom/pumpx2/cliparser/Main.java†L1-L208】
- `./gradlew :sampleapp:installDebug` builds and installs the example Android client (requires Android SDK setup).

## Module breakdown
### shared (`shared/src/main/java/com/jwoglom/pumpx2/shared`)
- Lightweight utilities used by both the Java protocol library and Android stack.
  - `Hex` supplies a platform-independent hex encoder/decoder to avoid Android `org.apache.http.legacy` conflicts.【F:shared/src/main/java/com/jwoglom/pumpx2/shared/Hex.java†L1-L45】
  - `JavaHelpers` centralizes reflection-based `toString` helpers and debugging utilities.【F:shared/src/main/java/com/jwoglom/pumpx2/shared/JavaHelpers.java†L1-L103】
  - `L` is a logging façade whose delegates can be rebound (e.g., to Timber in Android via `LConfigurator`).【F:shared/src/main/java/com/jwoglom/pumpx2/shared/L.java†L1-L64】【F:androidLib/src/main/java/com/jwoglom/pumpx2/util/timber/LConfigurator.java†L1-L44】
  - `TriConsumer`/`QuadConsumer` functional interfaces and the `LOG_PREFIX` constant support cross-platform logging.

### messages (`messages/src/main/java/com/jwoglom/pumpx2/pump/messages`)
- Implements the Tandem Bluetooth message model, packetization, and authentication primitives. Exposed as a pure-Java library (`pumpx2-messages`).
- **Message abstraction**
  - `Message` base class encapsulates cargo bytes, `MessageProps` metadata, and helpers for JSON/debug serialization. Requests and responses are annotated with `@MessageProps` describing opcode, cargo size, characteristic, signing, and API/device constraints.【F:messages/src/main/java/com/jwoglom/pumpx2/pump/messages/Message.java†L1-L120】【F:messages/src/main/java/com/jwoglom/pumpx2/pump/messages/annotations/MessageProps.java†L1-L26】
  - `MessageType` differentiates request vs response; opcodes are even/odd pairs by convention.【F:messages/src/main/java/com/jwoglom/pumpx2/pump/messages/MessageType.java†L1-L15】
  - `Messages` enum is the authoritative opcode registry. It binds request/response classes, registers them per-characteristic, and exposes helpers to instantiate or parse by opcode. Update this enum when adding new message pairs.【F:messages/src/main/java/com/jwoglom/pumpx2/pump/messages/Messages.java†L1-L223】【F:messages/src/main/java/com/jwoglom/pumpx2/pump/messages/Messages.java†L223-L394】
  - When introducing a new message, ensure its annotation references the counterpart class and add the pair to `Messages`. Stream responses (e.g., history logs) set `stream=true` to route through `StreamPacketArrayList`.
- **Packet handling & authentication**
  - `Packetize` converts a `Message` into BLE packets: adds opcode, transaction ID, length, optional HMAC-SHA1 signature (24-byte trailer), CRC16, and chunks into MTU-sized `Packet`s. It enforces `modifiesInsulinDelivery` gating via `PumpStateSupplier.actionsAffectingInsulinDeliveryEnabled`.【F:messages/src/main/java/com/jwoglom/pumpx2/pump/messages/Packetize.java†L1-L120】
  - `TransactionId` tracks the next transaction identifier (0–255) shared across the session.【F:messages/src/main/java/com/jwoglom/pumpx2/pump/messages/TransactionId.java†L1-L33】
  - `PacketArrayList` (and `StreamPacketArrayList` for streaming opcodes) reassembles multi-packet responses, validates CRC/signature, and enforces expected opcode/size/transaction ID. Use `PacketArrayList.build` to obtain the correct parser for a given message/characteristic.【F:messages/src/main/java/com/jwoglom/pumpx2/pump/messages/PacketArrayList.java†L1-L189】【F:messages/src/main/java/com/jwoglom/pumpx2/pump/messages/StreamPacketArrayList.java†L1-L95】

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jwoglom/pumpX2](https://github.com/jwoglom/pumpX2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
