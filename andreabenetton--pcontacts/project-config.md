---
trigger: always_on
description: Use this file as the default implementation context for this repo. Do not restate the architecture in every prompt — read it from ADRs and from the implementation plan. Optimize for correctness, security, reproducibility, and respect for the user's privacy stance.
---

# CLAUDE.md

## Purpose
Use this file as the default implementation context for this repo. Do not restate the architecture in every prompt — read it from ADRs and from the implementation plan. Optimize for correctness, security, reproducibility, and respect for the user's privacy stance.

---

## Repo stance
This repo is:
- **single-app Android** (Kotlin, Gradle, multi-module — see ADR-0011)
- **GPL-3.0-only**, with SPDX headers on every source file
- **F-Droid first**, sideload-friendly; **no Google Play Services, no telemetry, no proprietary blobs**
- **client-side-crypto-only**: the Proton API decrypt happens on-device; decrypted contact data is never logged and never sent off-device. It is written only to the system Contacts provider (the destination the user asked for); the app keeps no app-private plaintext copy — its own database holds content hashes and, for conflict detection, a per-contact last-known-server snapshot sealed under the device Keystore (ADR-0018)
- **unofficial-API consumer**: every claim about Proton's API is marked with `[V]` / `[U]` / `[A]` / `[D]` (verified / unverified / assumption / discouraged)

---

## Source precedence
For implementation work, use this order:

1. `docs/adr/NNNN-*.md` — architecture decisions. Contracts of the codebase.
2. The implementation plan (lives outside the repo at `/home/user/.claude/plans/act-as-a-staff-lovely-squid.md`) — the phased roadmap, risk register, and verification plan.
3. `NOTICE` — what we attribute and to whom.
4. The Proton/WebClients reference (`https://github.com/ProtonMail/WebClients` at the pinned commit recorded in `docs/API_RESEARCH.md` once that lands) — the executable specification we port from.
5. Existing code in this repo.

Rules:
- ADRs win for architecture. If existing code conflicts with an ADR, the code is wrong (or the ADR needs a superseding ADR).
- Proton's web client is the spec for protocol shape and crypto behavior; our Kotlin port must match it bit-exact where it must (see ADR-0013).
- An undocumented Proton behavior is **never** assumed safe — mark it `[U]` and design a fallback.

---

## Non-negotiable architectural rules

These are the load-bearing invariants. Every one corresponds to an ADR; read the ADR for the rationale.

### License + attribution (ADR-0001)
- License is **GPL-3.0-only**. Every source file carries an SPDX header.
- Files materially derived from ProtonMail/WebClients carry an additional `SPDX-FileCopyrightText` line crediting Proton AG, naming the upstream file, and pinning the upstream commit.
- New runtime dependencies must be GPL-3.0-compatible. The release build fails if they aren't (ADR-0015).

### Crypto strategy (ADR-0002, ADR-0013, ADR-0014)
- All crypto is **native Kotlin** in `:core:crypto`: BouncyCastle for OpenPGP, ported SRP-6a, ported bcrypt-SHA512.
- **No JS engine is bundled and no JavaScript runs for protocol or crypto work.** The only JavaScript the app ever executes is Proton's hosted captcha page, inside the locked-down WebView of ADR-0019.
- Every change to `:core:crypto` runs the captured-vector test suite (`tools/vectors/`) and must pass.
- The Proton SRP modulus signing public key is pinned (`core:crypto/src/main/resources/proton_srp_signing_key.asc`); modulus signature verification is mandatory before SRP arithmetic; on verification failure, login aborts.

### Decrypt client-side only (ADR-0007)
- Always pull encrypted `Cards[]` and decrypt locally.
- The app **never** calls `GET contacts/v4/contacts/export`. A CI grep fails the build if that path appears in any source file outside ADR-0007.
- Decrypted vCard bytes live only on the heap during a sync and in the system Contacts provider rows the sync writes; they are never logged, never transmitted off-device, and never stored app-privately in plaintext (Room holds hashes and the Keystore-sealed merge base of ADR-0018 only).

### Secrets storage (ADR-0009)
- All secret reads/writes go through the `SecretStore` interface in `:core:storage`.
- Direct `SharedPreferences` constructor calls outside `:core:storage` are forbidden (detekt rule).
- Every secret (tokens, `keyPassword`, the verification token) is sealed under the Keystore AEAD key `pcontacts.kekv1` before it touches the plain preferences file `pcontacts_auth_v2`; nothing in that file is readable without the key.
- Manifest invariants on release builds: `android:allowBackup="false"`, `android:debuggable="false"`. Asserted in a manifest-merger test.

### ContactsContract writes (ADR-0010)
- Every write to `RawContacts` / `Data` URIs uses `?caller_is_syncadapter=true`. The helper that builds these URIs is the **only** way to construct them in `:core:contacts-writer`.
- Update path is **delete-and-reinsert child `Data` rows** under a stable `RawContacts._ID`. Never delete the `RawContacts` itself on update (preserves user-owned aggregated state like starred / ringtone).
- `applyBatch` is chunked to ≤ 450 ops.

### No telemetry, no Google Services (ADR-0015)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andreabenetton/pcontacts](https://github.com/andreabenetton/pcontacts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
