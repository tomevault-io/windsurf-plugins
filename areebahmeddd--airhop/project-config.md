---
trigger: always_on
description: > This file is automatically loaded by VS Code GitHub Copilot for all sessions in this workspace. It provides the project context needed to give accurate, on-spec suggestions.
---

# Airhop: Copilot Workspace Instructions

> This file is automatically loaded by VS Code GitHub Copilot for all sessions in this workspace. It provides the project context needed to give accurate, on-spec suggestions.

## What This Project Is

**Airhop** is a React Native (Expo SDK 57, RN 0.86+, bare workflow, New Architecture) cross-platform iOS + Android application for **offline-first, private peer-to-peer communication** over Bluetooth mesh networks, with Nostr internet bridging and Cashu ecash payments.

It is **wire-protocol-compatible with bitchat** (`permissionlesstech/bitchat`, `permissionlesstech/bitchat-android`), both under the Unlicense (public domain). Airhop nodes and bitchat nodes communicate over BLE without configuration.

Both bitchat implementations are used as a reference checkout. They are **not part of this repository**: clone them into `bitchat/` yourself, which is where every path below expects them.

- `bitchat/ios/`: Swift iOS implementation (copy freely)
- `bitchat/android/`: Kotlin Android implementation (copy freely)
- `bitchat/georelays/`: relay discovery scripts and relay CSV

Nothing under `bitchat/` is committed, so never cite one of those paths in a file that ships with the repo. Read them freely; write the conclusion, not the citation.

## Read These Docs First

Before working on any code, read in this order:

1. [`docs/design/VISION.md`](docs/design/VISION.md): why + principles
2. [`docs/spec/ARCHITECTURE.md`](docs/spec/ARCHITECTURE.md): architecture, stack decisions, code snippets
3. [`docs/spec/PROTOCOLS.md`](docs/spec/PROTOCOLS.md): wire format and constants you must not break
4. [`docs/dev/PROGRESS.md`](docs/dev/PROGRESS.md): current build state

## Project Folder Structure

```
src/
  bridge/       # TurboModule TypeScript specs (Codegen input only)
  i18n/         # translation runtime + the bundled English catalog
  core/
    crypto/     # identity, keychain, noise-xx, noise-x, double-ratchet, contact-exchange
    mesh/       # wire/, routing/, links/, sync/, discovery/, rooms/, courier/, voice/
    nostr/      # nostr-client, courier-relay, gift-wrap, geo-relay, presence
    payments/   # cashu, nutzap
    router/     # transport selection
  services/     # long-lived runtime wiring, chiefly mesh-service
  features/     # screen-level logic (chat, contacts, wallet, discovery, settings)
  ui/           # shared components, theming
  store/        # Zustand slices + MMKV persistence
  platform/     # thin wrappers over OS APIs
  utils/        # pure helpers

android/        # Kotlin: BLE, WiFi Aware, voice, Tor modules + AirhopForegroundService
ios/            # Swift: BLE, WiFi Aware, pairing, voice, Tor modules
native/arti/    # Rust: the embedded Tor client both platforms compile

assets/data/    # nostr_relays.csv (bundled from bitchat/georelays/, CI-refreshed)
docs/
  design/       # VISION.md, ROADMAP.md
  spec/         # ARCHITECTURE.md, PROTOCOLS.md
  dev/          # PROGRESS.md, REFERENCE.md
.github/agents/ # specialized Copilot agents
.github/skills/ # domain reference files (read before working on a subsystem)
```

## Non-Negotiable Rules

Apply these to every suggestion, every file, every PR:

1. **All crypto = `@noble/*` only.** `@noble/curves` (X25519, Ed25519), `@noble/ciphers` (ChaCha20-Poly1305, XChaCha20), `@noble/hashes` (SHA-256, HMAC, HKDF). No other crypto library. No exceptions.

2. **Polyfill at entry point.** `import 'react-native-get-random-values'` must be the first import in `src/app/app.tsx` before any `@noble` import.

3. **Key storage.** Private keys via `src/core/crypto/keychain.ts` only (iOS Keychain / Android Keystore); never `expo-secure-store` directly, or the panic wipe cannot reach them. MMKV for non-secret state.

4. **Packet signing.** Every outgoing packet is Ed25519-signed. Every incoming packet has its signature verified before relay or display. Drop unsigned/invalid packets silently.

5. **No plaintext on disk.** Message content is encrypted at rest. Panic wipe destroys all keys, every database, the media cache and Tor state, and reports whether the keys actually went.

6. **Protocol compatibility.** Never change the bitchat v2 packet byte layout (`src/core/mesh/wire/packet-codec.ts`) or the BLE Service UUID without a version bump and compat test. See `docs/spec/PROTOCOLS.md`.

7. **Native code boundary.** Swift lives in `ios/`. Kotlin lives in `android/`. These expose **raw bytes** to TypeScript. Protocol logic, routing, and crypto decisions live in TypeScript (`src/core/`).

8. **Build order.** `src/core/` -> native modules -> `src/features/` -> `src/ui/`. Never write UI before core is unit-tested.

9. **Never hardcode user-facing text.** Add a key to `src/i18n/locales/en.ts` and use `T("your.key")` (component) or `t("your.key")` (outside React). The catalog keeps copy reviewable in one diff and is what makes a thirtieth language a new file rather than a sweep of every screen. CI fails on any hardcoded string. See [`i18n.md`](.github/skills/i18n.md).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [areebahmeddd/airhop](https://github.com/areebahmeddd/airhop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
