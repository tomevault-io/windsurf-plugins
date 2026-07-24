---
trigger: always_on
description: - [BBMOD_GML Coding Style and Naming Standards](#bbmod_gml-coding-style-and-naming-standards)
---

# BBMOD Development Guide

## Table of Contents

- [BBMOD_GML Coding Style and Naming Standards](#bbmod_gml-coding-style-and-naming-standards)
- [Scope](#scope)
- [Naming Conventions](#naming-conventions)
- [Documentation Style](#documentation-style)
- [GMLDoc Conventions (Required for New APIs)](#gmldoc-conventions-required-for-new-apis)
- [Formatting and Code Shape](#formatting-and-code-shape)
- [Section Divider Comments (Required)](#section-divider-comments-required)
- [Practical Rules for New Code](#practical-rules-for-new-code)
- [Production-Grade Implementation Quality](#production-grade-implementation-quality)
- [Runtime GML Reference (macOS)](#runtime-gml-reference-macos)
- [Notes on Existing Mixed Style](#notes-on-existing-mixed-style)
- [Alpha Branch and Changelog Policy](#alpha-branch-and-changelog-policy)
- [ChangelogDev.md Update Rules](#changelogdevmd-update-rules)
- [GameMaker Resource Registration (Exact Steps)](#gamemaker-resource-registration-exact-steps)
- [Files That Control Registration](#files-that-control-registration)
- [Important Repository-Specific Observation](#important-repository-specific-observation)
- [Preferred Workflow (Use GameMaker IDE)](#preferred-workflow-use-gamemaker-ide)
- [Manual Workflow (Deterministic)](#manual-workflow-deterministic)
- [Validation Checklist (Before Commit)](#validation-checklist-before-commit)
- [Common Failure Modes](#common-failure-modes)

## BBMOD_GML Coding Style and Naming Standards

### Scope

This file captures naming and style conventions observed in BBMOD_GML and should be treated as the default standard for new GML code and documentation.

### Naming Conventions

#### 1) Types and Constructors

- Struct and constructor names use `BBMOD_` + PascalCase.
- Examples: `BBMOD_Scene`, `BBMOD_Node`, `BBMOD_BaseRenderer`, `BBMOD_PointLight`.

#### 2) Global API Functions

- Public global functions use `bbmod_` + lower_snake_case.
- Examples: `bbmod_scene_get_current`, `bbmod_scene_set_current`, `bbmod_camera_get_zfar`, `bbmod_material_get`.

#### 3) Internal and Private Helpers

- Private globals and helpers use `__bbmod_` + lower_snake_case.
- Examples: `__bbmod_dll_is_supported`, `__bbmod_matrix_get_identity`.

#### 4) Struct Static Methods

- Preferred style in core runtime modules is lower_snake_case.
- Core evidence: `BBMOD_Scene` and `BBMOD_Node` static methods are lower_snake_case; `BBMOD_BaseRenderer` public methods are lower_snake_case and private internals use double-underscore lower_snake_case.
- Legacy modules may still expose PascalCase methods (notably collider and math-heavy areas). Keep compatibility there; do not introduce new mixed-style naming unless extending an existing mixed API.

#### 5) Enums and Enum Members

- Enum type names use `BBMOD_E` + PascalCase.
- Examples: `BBMOD_ECloudWeather`, `BBMOD_ERenderPass`.
- Enum members typically use PascalCase for named variants.

#### 6) Macros and Global Constants

- Macros and constants use uppercase snake case with a `BBMOD_` prefix.
- Internal macros use `__BBMOD_` uppercase snake case.
- Examples: `BBMOD_RELEASE_MAJOR`, `BBMOD_VERSION_MINOR`, `BBMOD_EV_ANIMATION_END`, `__BBMOD_BONE_SPACE_WORLD`.

#### 7) Variables and Parameters

- Function parameters use underscore prefix: `_deltaTime`, `_index`, `_scene`.
- Local temporary variables usually use underscore prefix: `_probe`, `_childCount`, `_matrix`.
- Short loop counters are commonly `i` without underscore.

#### 8) Fields and Properties on Struct Instances

- Public data fields use PascalCase.
- Examples: `AmbientLightColorUp`, `ReflectionProbes`, `FogIntensity`, `UseAppSurface`.
- Booleans typically use semantic prefixes such as `Is`, `Has`, `Enable`, `Use`.

### Documentation Style

- Use GMLDoc comments with triple slash: `///`.
- Common tags: `@module`, `@func`, `@var`, `@param`, `@return`, `@see`, `@private`, `@readonly`, `@desc`, `@note`.
- Keep docs close to the symbol they describe.

### GMLDoc Conventions (Required for New APIs)

These rules are based on observed usage in `BBMOD_GML/scripts` and should be followed for every new public API and struct member.

#### 1) Comment Prefix and Spacing

- Every doc line must start with `///` followed by a single space.
- Use `///` (empty doc line) to separate logical sections in a doc block.
- Never use `//////`, `///@tag` (missing space), or mixed plain comments inside a doc block.
- Inside constructors and static methods, keep doc indentation aligned with code indentation (tabs in this project).

#### 2) Line Length and Wrapping

Observed distribution across project `///` lines:

- `<= 80`: vast majority.
- `<= 100`: almost all lines.
- `> 100`: rare, mostly long `@func` signatures.

Required limits for new docs:

- Soft limit: 100 characters per `///` line.
- Hard limit: 120 characters for prose lines.
- Exception: `@func` signatures with many optional parameters may exceed 120 when needed.
- Wrap long prose across multiple `///` lines at phrase boundaries; do not split tokens or types.

#### 3) File-Level Block Order

At top of script files, use this order when applicable:

1. `@module`
1. Optional macro, interface, or enum docs (`@macro`, `@interface`, `@enum`, `@member`)
1. Constructor and global function docs (`@func` blocks)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [blueburncz/BBMOD](https://github.com/blueburncz/BBMOD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
