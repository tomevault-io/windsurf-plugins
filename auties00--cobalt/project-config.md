---
trigger: always_on
description: Cobalt is a Java implementation of the WhatsApp platform. It supports two independent transports behind one sealed API:
---

# Cobalt

Cobalt is a Java implementation of the WhatsApp platform. It supports two independent transports behind one sealed API:

1. **Linked clients** (`LinkedWhatsAppClient`) — reimplementation of WhatsApp Web/Desktop (companion, QR or pairing-code) and WhatsApp Mobile (primary, phone-number registration) over the encrypted binary-XMPP socket with Signal/Noise cryptography.
2. **Cloud API client** (`CloudWhatsAppClient`) — Meta's official WhatsApp Cloud API over `graph.facebook.com` REST plus an embedded webhook receiver.

## Build

- Maven project: `mvn compile` from root
- Java 25
- Module system with `module-info.java` in every Maven module

## Maven Modules

The reactor is a parent POM (`cobalt`) over the modules below. The wire-level domain (formerly the single `cobalt-model`) is split by wire FAMILY under the `cobalt-wire` aggregator. **Package == module name:** every wire/telemetry/stanza module's packages sit under a prefix equal to its JPMS module name (e.g. `cobalt-wire-linked` owns `com.github.auties00.cobalt.wire.linked.*`, `cobalt-telemetry-core` owns `com.github.auties00.cobalt.telemetry.*`).

| Module                    | artifactId           | JPMS name                                 | Purpose                                                                                                                                                                                                                                             |
|---------------------------|----------------------|-------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `modules/wire`            | `cobalt-wire`        | (aggregator POM, no JPMS module)          | Parent of the seven wire modules below; no code.                                                                                                                                                                                                   |
| `modules/wire/wire-core`  | `cobalt-wire-core`   | `com.github.auties00.cobalt.wire.core`    | Shared transport-agnostic wire contracts and leaf types: the `Jid` addressing family, protobuf mixins, leaf utilities, and the message envelope/key/status/content contracts both transports implement. Depends only on protobuf + libsignal.       |
| `modules/wire/wire-linked`| `cobalt-wire-linked` | `com.github.auties00.cobalt.wire.linked`  | Hand-written protobuf domain model for the Linked transport (messages, chats, contacts, newsletters, calls, business/commerce, settings, app-state/syncd, Signal/pairing/device-fanout). Every type is `@ProtobufMessage`/`@ProtobufEnum`/`@ProtobufMixin`. |
| `modules/wire/wire-cloud` | `cobalt-wire-cloud`  | `com.github.auties00.cobalt.wire.cloud`   | Meta Cloud API JSON request/response and webhook models, plus the Cloud-native message envelope and content bodies. Implements the wire-core contracts; does NOT depend on wire-linked.                                                             |
| `modules/wire/wire-wam`   | `cobalt-wire-wam`    | `com.github.auties00.cobalt.wire.wam`     | WAM `@WamEvent`/`@WamEnum` schema declarations, consumed by the `cobalt-wam-core` processor. Separate from wam-core because a processor cannot process its own sources.                                                                             |
| `modules/wire/wire-stanza`| `cobalt-wire-stanza` | `com.github.auties00.cobalt.wire.stanza`  | Typed IQ/MEX/SMAX/USync stanza operation models (the query schema).                                                                                                                                                                                |
| `modules/wire/wire-graphql`| `cobalt-wire-graphql`| `com.github.auties00.cobalt.wire.graphql` | Facebook/WhatsApp/WhatsApp Web GraphQL operation models.                                                                                                                                                                                           |
| `modules/wire/wire-push`  | `cobalt-wire-push`   | `com.github.auties00.cobalt.wire.push`    | Third-party push-protocol wire types (Google FCM MCS stanzas, Android check-in) the mobile registration flow speaks.                                                                                                                               |
| `modules/stanza-core`     | `cobalt-stanza-core` | `com.github.auties00.cobalt.stanza`       | The `Stanza` node model (`stanza.model`) and its binary-XMPP tokenization codec (`stanza.binary`), plus the `SizedInputStream` sized-payload leaf.                                                                                                 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Auties00/Cobalt](https://github.com/Auties00/Cobalt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-19 -->
