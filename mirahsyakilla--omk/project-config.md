---
trigger: always_on
description: - Before adding a method, helper, abstraction, configuration field, or utility, inspect the
---

# OhMyKeymint Agent Guide

## Project Boundaries

- Before adding a method, helper, abstraction, configuration field, or utility, inspect the
  directly relevant modules for equivalent behavior. Reuse or minimally extend existing code.
- Android is the only production and acceptance target. Target-specific build, check, and test
  commands must use `aarch64-linux-android`; host or x86_64 success alone is insufficient.
- Compatibility scope: Android 12-17; Linux kernels 4.14-6.18 and newer LTS kernels. Preserve
  behavior across this range, and do not claim a platform was validated without evidence.
- Do not introduce new non-Rust product or runtime code without explicit approval. Existing Python
  build/deployment scripts and shell packaging assets are tooling, not precedent for new runtime
  components.

## Review Scope

- Do not report or fix scenarios that require multiple independently abnormal, low-probability
  conditions and are unreachable through normal supported operation.
- When excluding such a scenario, state the concrete conditions that must coincide and why normal
  supported paths cannot reach it. Low frequency or a narrow timing window alone does not exclude
  an issue reachable through normal supported operation.

## Behavioral Invariants

### OMK Routing

- For every request routed by `scoop` with `FilterDecision::allowed == true`, OhMyKeymint is the
  only backend during normal reachable operation. Per-method intercept settings still determine
  whether a request is routed by `scoop`.
- A scooped `generateKey`, `importKey`, `importWrappedKey`, `createOperation`, `deleteKey`,
  `getKeyEntry`, `updateSubcomponent`, `grant`, `ungrant`, `listEntries`, `listEntriesBatched`,
  and `getNumberOfEntries` stays on OhMyKeymint. A miss, or a keyblob OhMyKeymint cannot open, is
  returned as an OhMyKeymint error. It is not sent to the real TEE, and list results do not include
  real TEE aliases. A stored patch level ahead of the current HAL patch does not block use of that
  key.
- `generateKey` rejects an illegal purpose with `IncompatiblePurpose`: EC `Encrypt`, `Decrypt`,
  and `WrapKey`; RSA `AgreeKey`; any ML-DSA purpose other than `Sign`, `Verify`, and `AttestKey`.
- Choose any other backend only from the current caller, filter decision, method, and configuration.
  Do not read or infer which backend created a `KEY_ID`, `GRANT`, wrapping key, or attestation key.
- Per-method intercept settings are authoritative. When interception for a method is disabled, pass
  the request to System unchanged even for a caller allowed by `scoop`
- Do not support key or descriptor continuity between System and OMK. Pass old, externally
  supplied, and System-created descriptors to the selected backend unchanged; an OMK business
  error for such a descriptor is authoritative.
- Return OMK business errors as-is. Reachable transport, boundary, or injector failures that are
  not OMK-unavailable must not fall back to a successful system reply; normalize them to the
  existing AOSP-compatible error path, such as `SYSTEM_ERROR` where applicable.
- Only confirmed OMK-unavailable failures may preserve the original system reply. These include a
  missing service/backend, connection failure, and stale or dead RPC transport failures such as
  `DeadObject`, `RpcError`, and `NotEnoughData`. Reuse the existing classifiers instead of
  maintaining another status list.

### Persistent and Temporary Files

- Obtain the user's explicit approval before creating any file that requires permanent storage.
- Delete every temporary probe artifact immediately after the probe completes.
- All persistent data should be in `/data/misc/keystore/omk/data/`

### Telephony Attestation IDs

- `AttestationIdInfo` is field-wise. A device may legitimately have one IMEI, no IMEI2, no MEID,
  or no telephony identifiers. Empty optional telephony fields, including explicit empty overrides,
  do not invalidate brand, device, product, serial, manufacturer, model, or an available IMEI.
- Runtime discovery is best-effort and one-shot per KeyMint process once the required Binder
  services are registered; TEE and StrongBox share the resolved snapshot. After all applicable
  direct Binder APIs and property fallbacks have been attempted once, cache and return
  `Some(AttestationIdInfo)` even when telephony fields are partial or all empty.
- If an actually attempted `phone`, `iphonesubinfo`, or `package_native` Binder service is not yet
  registered, return `Ok(None)` without updating runtime, persisted, process, or TA caches so a
  later request can retry. Empty values, unsupported APIs, permission failures, and other probe
  errors from registered services still complete the one-shot snapshot.
- `Ok(None)` means the entire ID snapshot is not ready and remains retryable. Do not use it merely
  because IMEI2, MEID, or all telephony fields are absent after one-shot discovery.
- Preserve every valid candidate returned by any slot or API. A failure or unsupported result from
  another probe must not discard successful values or turn the resolved snapshot into an error.
  Individual probe failures are internal discovery failures, not scoop-routed OMK business errors.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MirahSyakilla/OMK](https://github.com/MirahSyakilla/OMK) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
