---
trigger: always_on
description: Top-level skill for Claude Code sessions across any peat-* repo. Read first, then read the per-repo SKILL.md.
---


# Peat Ecosystem SKILL

Peat is an interoperability-first mesh registry sync platform built for heterogeneous autonomous systems in defense and edge environments. Its core value proposition is **interoperability that enables scale** — Peat connects systems that don't speak the same language across transport and protocol boundaries. Peat is developed under the Defense Unicorns GitHub org: https://github.com/defenseunicorns

## When this skill applies

- Any session touching files in a `peat-*` repo or the top-level `peat` crate
- Cross-repo changes affecting more than one peat repo
- Reviewing a PR in any peat repo

After reading this file, read the relevant per-repo SKILL.md from the router below. Per-repo skills are authored against `peat/SKILL_TEMPLATE.md`.

## Skill router

Read only what's relevant to the current task. Do not preload every per-repo skill.

The ecosystem comprises **one Rust workspace repo** (`peat`) plus several sibling repos. Internal workspace subcrates do **not** have separate per-repo skills — they share this file.

**Sibling repos** (each its own git repo, each with its own per-repo skill):

| Repo | Purpose | Skill |
|---|---|---|
| **peat** | Rust workspace. Hosts the ecosystem skill (this file) and the per-repo skill for the workspace. Active. | This file |
| **peat-mesh** | Mesh networking; pluggable transport, peer discovery, topology, routing, optional Automerge/Iroh sync, optional HTTP/WS broker. **Top-tier active.** | `peat-mesh/SKILL.md` |
| **peat-btle** | BLE transport bridge. M5Stack/ESP32 integration. | `peat-btle/SKILL.md` |
| **peat-lite** | Lightweight implementation for constrained environments. | `peat-lite/SKILL.md` |
| **peat-atak-plugin** | One consumer-plugin integration (Android/Kotlin). Pure Kotlin/Gradle — no Rust here. Consumes pre-built JNI/UniFFI bindings from `peat/peat-ffi`. Repo name is historical; this repo treats it as a generic consumer surface. | `peat-atak-plugin/SKILL.md` |
| **peat-sim** | ContainerLab-based network simulation harness. Not on the production path. | `peat-sim/SKILL.md` |

**Workspace subcrates** (members of the `peat` repo, share this skill):

| Subcrate | Role |
|---|---|
| `peat` | Top-level crate; eventual home for shared types and traits. Currently a placeholder (see "`peat` repo-specific skill" below). |
| `peat-schema` | Schema definitions. |
| `peat-protocol` | Protocol logic (`Translator`, `ChangeEvent`, etc.). |
| `peat-transport` | Transport abstractions used by the workspace. |
| `peat-persistence` | Persistence layer. |
| `peat-ffi` | FFI bindings for Kotlin/Swift (UniFFI 0.28, proc-macro mode) + direct `jni 0.21` for Android. The only routinely-unsafe Rust in the ecosystem. |

**Status unknown — confirm with Kit before authoring skills.** The following were listed in earlier drafts but aren't currently visible from this checkout: `peat-registry`, `peat-node`, `peat-gateway`, `peat-rmw`, `peat-mavlink`. They may be planned, renamed, deprecated, or in private repos. Per the active-repos record, only `peat` and `peat-mesh` are currently top-tier active.

## Hard invariants (cross-cutting)

These rules apply in every repo. Violating one without explicit user approval is out of scope, full stop.

**Language.** Rust everywhere except Kotlin in consumer plugins (currently `peat-atak-plugin`). No new language dependencies. No Python. No shell scripts for anything that belongs in Rust.

**FIPS-approved cryptographic primitives only.** Every algorithm used anywhere in the peat ecosystem must be on the FIPS 140-3 approved list. AEAD: AES-GCM (not ChaCha20-Poly1305). Signatures: Ed25519 (FIPS 186-5) or ECDSA-P256/P384. Key agreement: ECDH-P256/P384 (X25519 only with explicit review). KDF: HKDF-SHA-2. MAC: HMAC-SHA-2. TLS/QUIC must run under a FIPS-mode crypto provider (e.g. `aws-lc-rs` for rustls; the default `ring` is **not** FIPS-validated). MLS suites must be FIPS-aligned (e.g. `MLS_128_DHKEMP256_AES128GCM_SHA256_P256`). Existing ChaCha20-Poly1305 references in ADR-006/044/048/049 + spec docs are tracked for amendment; do not propagate them. Canonical reference: ADR-060 §5 "Cryptographic primitives (FIPS posture)" + `CLAUDE.md` § "Hard rule: FIPS-approved cryptographic primitives only".

**No consumer-specific references in peat.** peat is the generic mesh substrate; consumers (mobile-app plugins, wearable firmware, CLI tools, server bridges) live in sibling repos. Code, comments, examples, READMEs, operational docs, JNI symbol names, package paths, and test fixtures in this repo MUST NOT name a specific consumer (ATAK, WinTAK, iTAK, WearTAK, etc.) or vendor-derived identifiers. Use "consumer", "consumer plugin", "CoT consumer", "mobile-app plugin", "wearable", "CLI tool", or "server bridge". Protocol-name appearances (e.g. CoT, TAK Server wire protocol) are allowed where the protocol itself is structurally load-bearing; consumer-name appearances are not. The only exception is ADRs in `docs/adr/` citing real-world use cases that motivated a design decision. See `CLAUDE.md` § "Hard rule: no consumer-specific references in peat" for the full rule + rationale. Verification gate (below) includes a grep check for ATAK / vendor names in new diffs.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [defenseunicorns/peat](https://github.com/defenseunicorns/peat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
