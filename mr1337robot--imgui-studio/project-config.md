---
trigger: always_on
description: This repository contains ImGui Studio, a local-first, AI-native development environment for authoring polished, animated Dear ImGui menus in real C++.
---

# AGENTS.md

## Purpose

This repository contains ImGui Studio, a local-first, AI-native development environment for authoring polished, animated Dear ImGui menus in real C++.

Agents working in this repository must preserve the defining product promise:

> Project menu and widget source is compiled into the real Dear ImGui WebAssembly preview and the same source is exported for native C++ integration.

This file applies to the entire repository. A more deeply nested `AGENTS.md` may add directory-specific instructions, but it must not weaken the product, security, determinism, parity, or testing requirements defined here.

## Current repository state

The repository is currently in the specification and architecture phase. The root documentation is the implementation baseline. Do not treat example paths, APIs, schemas, or acceptance criteria as aspirational prose; they are contracts unless explicitly marked post-MVP or deferred.

Before implementing a feature, confirm that its required parent directories and build foundations exist. Do not fabricate successful build or test results when the corresponding implementation or toolchain does not exist yet.

## Required reading

Read the documents relevant to the task before making changes. For foundational or cross-cutting work, read them in this order:

1. `PRD.md` — product scope, users, requirements, and definition of done.
2. `TECHNICAL_DESIGN.md` — processes, boundaries, technology choices, lifecycle, and data flow.
3. `MVP_IMPLEMENTATION_PLAN.md` — dependency order, work packages, phase gates, and release blockers.
4. The contract document governing the area being changed.
5. Relevant records in `docs/adr/`.

Area-specific contracts:

| Area | Required documents |
|---|---|
| C++ runtime or custom widgets | `RUNTIME_API.md`, `ANIMATION_SPEC.md`, `INSPECTION_PROTOCOL.md` |
| Deterministic time, replay, or capture | `ANIMATION_SPEC.md`, `PROJECT_FORMAT.md`, `AGENT_TOOL_API.md` |
| Agent tools or local HTTP/MCP API | `AGENT_TOOL_API.md`, `PROJECT_FORMAT.md`, `INSPECTION_PROTOCOL.md`, `SECURITY_MODEL.md` |
| Project files, manifests, assets, or scenarios | `PROJECT_FORMAT.md`, `SECURITY_MODEL.md` |
| Inspection or diagnostics | `INSPECTION_PROTOCOL.md`, `RUNTIME_API.md`, `TEST_PLAN.md` |
| Browser preview or build pipeline | `TECHNICAL_DESIGN.md`, `SECURITY_MODEL.md`, ADRs 0001 and 0003 |
| Native export or integration | `EXPORT_AND_INTEGRATION.md`, `TEST_PLAN.md`, ADRs 0002 and 0005 |
| Security-sensitive work | `SECURITY_MODEL.md`, `AGENT_TOOL_API.md`, `PROJECT_FORMAT.md` |
| Tests, fixtures, benchmarks, or release work | `TEST_PLAN.md`, `MVP_IMPLEMENTATION_PLAN.md` |

Do not partially read a selected contract and then infer the rest. Public API and protocol details are intentionally spread across the relevant documents and cross-links.

## Contract precedence

When documents appear inconsistent, use this precedence:

1. Accepted ADR for the specific decision.
2. `PRD.md` for product scope and user-visible requirements.
3. `TECHNICAL_DESIGN.md` for system architecture.
4. Area-specific contract documents for exact API, schema, lifecycle, and error behavior.
5. `MVP_IMPLEMENTATION_PLAN.md` and `TEST_PLAN.md` for work order and verification.

Do not silently choose one side of a real contradiction. Identify the conflict, determine whether it changes an accepted decision, and update all affected documents in the same change. Add or supersede an ADR when changing a consequential architectural decision.

## Non-negotiable MVP decisions

All implementation work must preserve these decisions:

- The canonical browser preview runs real Dear ImGui compiled from C++ to WebAssembly.
- C++20 is the project authoring and export language for the MVP.
- Browser preview and native export compile the same project menu and widget source.
- The initial browser renderer is WebGL2 through the pinned Emscripten/OpenGL3 path.
- The MVP is local-first, single-user, and Windows-native for parity and export fixtures.
- Dear ImGui, Emscripten, Studio runtime, schemas, and protocols are explicitly versioned.
- Custom widgets and `ImDrawList` rendering are primary capabilities, not edge cases.
- Selected internal Dear ImGui behavior is isolated behind a versioned adapter where practical.
- Direct `imgui_internal.h` use is allowed but must be detected and reported as a portability concern.
- Deterministic animation time uses signed 64-bit integer microseconds at protocol boundaries.
- Deterministic captures start from a clean reset and use stable input ordering.
- Screenshots are paired with structured widget, geometry, interaction, animation, and diagnostic data.
- Stable Studio widget identifiers are distinct from incidental pointers or screen coordinates.
- Exports are produced from an immutable successful, smoke-passed build revision.
- A failed build never replaces the last successful preview.
- The portable rendering tier is the only tier shipped in the MVP.
- Enhanced blur, bloom, and custom render passes are post-MVP and must not leak into portable projects.
- Browser/native geometry tolerance is at most two pixels at the fixed benchmark configuration.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mr1337robot/imgui-studio](https://github.com/mr1337robot/imgui-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
