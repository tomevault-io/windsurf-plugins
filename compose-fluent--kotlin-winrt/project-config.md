---
trigger: always_on
description: `kotlin-winrt` is a Kotlin language projection for WinRT and WinUI, and its engineering baseline is the local `.cswinrt/` source tree in this repository.
---

# AGENTS

## Mission

`kotlin-winrt` is a Kotlin language projection for WinRT and WinUI, and its engineering baseline is the local `.cswinrt/` source tree in this repository.

The default expectation is not to invent a new projection model. When implementing runtime behavior, generated bindings, authoring support, activation, marshaling, delegates, generic interface handling, collection projection, or WinUI bootstrap behavior, use `.cswinrt` as the first reference and keep Kotlin behavior structurally aligned with it.

## Primary Reference Source

1. Treat `.cswinrt/` as the primary source of truth for architecture, layering, naming intent, feature slicing, and projection behavior.
2. Before introducing a new runtime abstraction or generator rule, inspect the corresponding area in `.cswinrt/src` and mirror its responsibility split in Kotlin form.
3. If `kotlin-winrt` behavior differs from `.cswinrt`, prefer changing `kotlin-winrt` to match the reference unless there is a Kotlin-specific language or toolchain constraint that makes direct parity impossible.
4. If parity is impossible, document the exact reason in the relevant code or task notes and keep the deviation narrow, explicit, and test-covered.
5. Do not design public APIs, runtime conventions, or generator heuristics from scratch when `.cswinrt` already has a corresponding implementation strategy.

## Design Direction

1. The implementation direction is reference-first and design-first: inspect the matching `.cswinrt` slice, decide the Kotlin module boundary and runtime contract, then implement the Kotlin version, and only after that add or adjust tests as validation.
2. Tests are validation tools, not design drivers. Do not infer missing architecture, API shape, marshaling rules, activation behavior, or generator policy by iterating on failing tests until something passes.
3. Do not use test failures as the primary source of truth for what the runtime or generator should do. Use `.cswinrt` source and the planned module responsibilities as the primary source of truth, then use tests to confirm parity.
4. If an existing test contradicts `.cswinrt`, fix the implementation or the test so that the repository moves toward `.cswinrt` parity rather than preserving a Kotlin-only behavior.
5. Before writing a new test for a new slice, identify the matching `.cswinrt` source area and encode that mapping in code comments, task notes, or `PLAN.md` status text when it is not already obvious.
6. Do not let ad hoc test scaffolding become the place where design decisions are made. Runtime contracts belong in `winrt-runtime`, metadata/model decisions belong in `winrt-metadata`, generator decisions belong in the generator pipeline, and tests only verify those decisions.

## Execution Cadence

1. Work must advance in explicit phases rather than opportunistic file-by-file edits.
2. Phase 1 is runtime-first: implement the minimum ABI, activation, marshaling, object identity, and interface-call foundations in `winrt-runtime` that are required by the current `.cswinrt` slice.
3. Phase 2 is metadata-second: implement WinMD loading, metadata normalization, symbol modeling, and projection-shape inputs in `winrt-metadata` only after the active runtime contracts are clear enough to support the slice being built.
4. Phase 3 is generator-third: implement generator rules from the `.cswinrt/src/cswinrt` responsibility split only after the required metadata model and runtime contracts exist.
5. Phase 4 is projections-fourth: generate or check in projection output in `winrt-projections` only after the corresponding generator rule is defined and the runtime/metadata contracts it depends on already exist.
6. Phase 5 is authoring-fifth: implement `winrt-authoring` only after the runtime ABI/lifetime and metadata/generator contracts needed by the authoring slice are understood from `.cswinrt/src/Authoring`.
7. Phase 6 is samples-and-validation last: use `winrt-samples` and per-module tests only after the corresponding runtime, metadata, generator, projection, or authoring slice has already been designed and implemented.
8. Do not start from `winrt-projections`, sample code, or test scaffolding just because they are easier to see or quicker to make compile. If a slice appears to require projection output first, stop and fill the missing upstream runtime, metadata, or generator step instead.
9. A later phase may contain temporary smoke coverage for an earlier phase, but it must not become the reason the earlier phase is designed backward from that coverage.
10. If a phase is blocked, record the missing prerequisite in `PLAN.md` and continue with the earliest incomplete prerequisite rather than jumping ahead to a later module.

## Required Slice Ordering

Within the phase order above, the active implementation queue must also move from foundational slices to dependent slices:

1. In `winrt-runtime`, prioritize ABI primitives, HRESULT/GUID/HSTRING ownership, initialization scope, object identity, activation lookup, and reusable vtable-call shapes before delegates, collections, async, or WinUI-specific conveniences.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [compose-fluent/kotlin-winrt](https://github.com/compose-fluent/kotlin-winrt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
