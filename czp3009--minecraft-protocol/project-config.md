---
trigger: always_on
description: This file contains repository-wide rules for coding agents. The root [README.md](README.md) is the human-facing project
---

# Agent development guide

This file contains repository-wide rules for coding agents. The root [README.md](README.md) is the human-facing project
guide. Before changing a directory, read its nearest `AGENTS.md`; a nested guide may add local ownership, invariants,
and verification steps, but must not repeat this file.

## Authority and design goals

- Treat checked-in source, build scripts, generated-source wiring, and tests as the authority for the current project.
  Keep documentation and agent guidance aligned with them.
- This is an early-stage project. Prefer a coherent design over compatibility shims, deprecated aliases, transitional
  paths, or preserving an accidental module boundary.
- Keep the high-level vanilla client and server paths zero-configuration beyond facts the application must supply, such
  as a selector, remote address, player identity, or semantic values omitted by the selected representation but needed
  to construct an in-memory value. Connection definitions, negotiation profiles, protocol data, and vanilla data-pack
  registry projectors have release-matched defaults; mods remain explicit overrides or extensions.
- Inspect existing code and tests before editing. Preserve unrelated work in a dirty worktree and change the layer that
  owns the behavior.
- Gameplay, authoritative world ticking, permission systems, persistence policy, and a general-purpose Minecraft server
  are outside this library's scope.
- Runtime libraries provide data and conversion capabilities only; there is no library-owned application layer.
  Keep concrete game-content bindings and algorithms, such as chest/hopper/furnace state and merchant rules, in test
  source sets. `demo` applications are the only production-source exception. Protocol packet schemas, saved-file
  schemas and generated official data remain format/data contracts, not gameplay implementations.

## Module boundaries

Runtime libraries are arranged from reusable formats and models toward connection orchestration:

| Module                           | Responsibility                                                                          |
|----------------------------------|-----------------------------------------------------------------------------------------|
| `nbt`                            | Format-independent NBT values and the logical serializer handoff                        |
| `nbt-serialization`              | Binary NBT, SNBT, and their `kotlinx.serialization` formats                             |
| `protocol-model`                 | Packet payloads, shared protocol values, logical serializers, and wire annotations      |
| `protocol-serialization`         | Physical packet payload encoding and packet registries                                  |
| `protocol-world`                 | Directional conversion between semantic world values and packets                        |
| `protocol-configuration`         | Configuration projection, received registry views and dimension resolution              |
| `protocol-configuration-vanilla` | Generated official registries and Configuration defaults                                |
| `datapack-vanilla`               | Generated official pack archives, parsed packs and stack completion                     |
| `protocol-transport`             | Ktor sockets, framing, compression envelopes, and stream encryption                     |
| `protocol-session`               | Typed packet dispatch, direction, state transitions, and loader negotiation profiles    |
| `distribution-metadata`          | Modern version, asset-index, and Java runtime metadata plus streaming downloads         |
| `account-auth`                   | Launcher-side Microsoft, Xbox, and Minecraft Services HTTP APIs                         |
| `protocol-auth`                  | Game identities, Session/Services HTTP APIs, Login cryptography, and chat signing       |
| `protocol-client`                | Client orchestration through entry into Play plus received world projections            |
| `protocol-server`                | Server orchestration through entry into Play plus finite initial-view projection        |
| `world-format`                   | Filesystem-independent world schemas, data packs, Anvil containers, and semantic chunks |
| `world-io`                       | Okio paths, world leases, files, and filesystem-backed stores                           |

Use the representation-stage names consistently across module boundaries: `DataPackArchive` is raw file bytes,
`DataPack` is parsed content, `DataPackStack`/`ResolvedDataPackStack` are priority views, `ResolvedConfigurationData` is
the
server-side Configuration projection, `DataPackConfigurationSnapshot` is the client-visible capture, and
`ClientRegistryView` is its resolved lookup view. Variables use the corresponding lower-camel name where the full type
name remains readable.

Private development infrastructure has separate boundaries:

- `buildSrc` owns shared Gradle configuration, official-artifact preparation and analysis, non-source-input generators,
  and Fixture Host service wiring.
- `protocol-symbol-processor` owns KSP generation derived from source annotations.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [czp3009/minecraft-protocol](https://github.com/czp3009/minecraft-protocol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
