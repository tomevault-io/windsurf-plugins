---
trigger: always_on
description: This is the first gate. It outranks shipping speed, local convenience, and default agent helpfulness. A change that invents a parallel helper, parser, protocol model, transport, or dependency without a recorded reuse search fails, even if tests pass. Do not start coding until the search below is done. "I will reuse later" is a failure.
---

# Any AI CLI Remote Engineering Rules

## Reuse is a quality gate

This is the first gate. It outranks shipping speed, local convenience, and default agent helpfulness. A change that invents a parallel helper, parser, protocol model, transport, or dependency without a recorded reuse search fails, even if tests pass. Do not start coding until the search below is done. "I will reuse later" is a failure.

- Search for an existing implementation before adding one. Reuse the canonical helper or extend it instead of copying its logic.
- Before writing non-trivial infrastructure, search the relevant official SDK, package registry, and maintained open-source projects. The required order is: reuse repository code, use the platform or language standard library, adopt a compatible maintained library, and only then write the smallest missing adapter.
- That search is mandatory, not optional review advice. Check current upstream documentation and releases online before coding; stale memory of a library or version is not sufficient evidence.
- A custom implementation is allowed only when the dependency decision records why the existing libraries are incompatible, unmaintained, incorrectly licensed, unsafe, or materially larger than the missing behavior. "Faster to write" is not a valid reason.
- Do not reimplement a published protocol with hand-written duplicate wire types when an official or established SDK covers them. Provider extensions must wrap or extend the shared SDK rather than fork its model.
- Prefer actively maintained libraries with tests, documented releases, and an open-source license compatible with this repository. Do not add a dependency for behavior already covered clearly by the standard library.
- Record each non-obvious dependency/reuse decision in `docs/DEPENDENCY_DECISIONS.md`, including the candidates checked, the chosen implementation, and the scope that must not be duplicated.
- A new custom parser, renderer, protocol model, process enumerator, persistence format, crypto helper, or transport fails review unless the decision log names the maintained libraries checked and gives a concrete incompatibility for each rejected candidate.
- There must be one shared implementation for each cross-provider concern, including registry behavior, session metadata, canonical path containment, timestamp normalization, pagination, connection generations, and compatibility migration.
- Adapters contain only provider-specific behavior. If code does not depend on a provider's protocol or on-disk format, it does not belong in the adapter.
- Do not create near-duplicate helpers with different names. Consolidate them and update callers.
- Prefer real module boundaries during architecture and refactoring: layer cohesive responsibilities such as model, transport, storage, domain, feature, and composition rather than merely splitting files while retaining one monolithic module.
- Put each shared capability in one owning module and expose the narrow public API required by its consumers. Do not scatter or duplicate helpers, protocol parsing, storage, networking, or UI orchestration across modules; consolidate scattered responsibilities before extending them.
- Avoid over-modularization as well: do not create a fragment module or wrapper without an independent responsibility or a real second caller.
- A facade whose methods only forward to another type is not a split. The 600-line gate is failed by that wrapper, not satisfied by it. Move the logic; do not add `Coordinator` / `Controller` / `Scope` / `Effects` types to keep a file under the limit.
- Transcript follow is list behavior: pin to the latest item while following, pause on user drag, resume on the jump-to-bottom control. One owner per client, same behavior on Android and iOS. Do not add a FollowController, EffectsState, transcript-hash `snapshotFlow`, spacer anchor item, or per-widget scroll calls.
- Backward-compatibility reads and migrations must be centralized. New writes use only the current Any AI CLI Remote names.

## Current product

This repository is past the bootstrap. It started as a Grok-branded remote and is now **Any AI CLI Remote**: a provider-neutral Go daemon, a macOS launcher, and native Android/iOS chat clients. Grok is the first Provider adapter, not the product name. Do not restart items 0–6. Do not scaffold a second Provider. Do not treat leftover Grok Remote identifiers as the current brand.

Shipped native surfaces (Android and iOS, unless noted):

- Device pairing: `anyaicliremote://pair` QR/deep link, Keychain/Keystore, multi-device profiles, disconnect-and-return. Pairing never chooses a workspace.
- macOS launcher: device name, daemon/provider ports, bind and optional public host, start/stop of the owned daemon, pairing QR. No workspace picker.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rezoch340/any-aicli-remote](https://github.com/rezoch340/any-aicli-remote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
