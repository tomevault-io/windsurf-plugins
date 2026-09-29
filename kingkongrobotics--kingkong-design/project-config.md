---
trigger: always_on
description: The public project name is **Design Workflow**. Appearance `.skin` and environment `.map` creation are equal workflows. First identify the requested route. For maps, gather scene requirements, scale, terrain/props, challenge targets, robot profile, spawn and visual acceptance; use scene authoring/build/export rather than the print task ledger. For display-only appearance packaging, use the existing complete-robot assembly and `export-skin`. Apply CAD installation, print engineering, slicing and 
---

# Instructions for AI assistants working in this repository

## Project scope and request routing

The public project name is **Design Workflow**. Appearance `.skin` and environment `.map` creation are equal workflows. First identify the requested route. For maps, gather scene requirements, scale, terrain/props, challenge targets, robot profile, spawn and visual acceptance; use scene authoring/build/export rather than the print task ledger. For display-only appearance packaging, use the existing complete-robot assembly and `export-skin`. Apply CAD installation, print engineering, slicing and physical-fit rules only to physical-shell work. APP execution and website implementation are outside this repository. Keep established CLI, protocol and historical provenance identifiers compatible.

## Required design selection before modeling

For every new appearance, including display-only skins and physical shells, present distinct design candidates and wait for the user's explicit choice before selecting a modeling tool or generating 3D geometry. Do not treat silence, a default option, or an assistant preference as a selection. Reuse a design or existing model explicitly selected by the user in the current task; record that instruction without asking again. Pure repackaging or validation of an unchanged existing asset does not require new concept selection. Scene authoring remains a separate route.

Record the selected design, actual user confirmation, and immutable reference hashes as described in docs/workflow.md. Only then assess available tools and explain the suitable route using docs/providers.md. Tripo is optional; never require an account or assume permission to spend credits. If available tools cannot preserve the selected design, explain the limitation and obtain a new choice before simplifying or substituting it.

Review actual front, side, back and assembled color renders against the chosen silhouette, proportions, details and palette. Also inspect gray geometry and activity clearances. Iterate on mismatches before delivery; package validity is not visual acceptance. Never present concept art as a render of the generated model.

## Required appearance and scene exchange protocol (v0.6)

Read the [content package protocol](docs/content-packages.md) first. Deliver appearances as `.skin` and scenes as `.map`. Do not invent another directory layout, unit system, manifest field, or extension. For new production use `export-skin` / `export-map`; run `verify-package` before delivery, passing the selected trusted `--profile` for skins. Use `--mujoco` to check actual loading. Package parsing, platform compatibility, simulation loading, manufacturing checks, and physical fitting are separate conclusions.

- `.skin/3` is display only. It must contain a complete whole-robot URDF/MJCF and relative display meshes, and must not contain print STL, AMS 3MF, G-code, or STEP. Deliver print files separately, bound to the exact mechanical interface and profile SHA. Undeclared parts retain the baseline configuration. Never fabricate a whole robot from one shell or treat different mechanical revisions as interchangeable.
- For whole-robot color requests, use `assemble --visual-overrides FILE.json` or a batch recipe in `library/skin-recipes.json`; declare lower-shell and limb visual RGBA in `.skin/3` `visual_overrides`. Change only explicitly declared, noncollision visual parts, never the baseline profile or physical parameters. Legacy `/1` and `/2` are read-compatible only.
- New `.map` production uses `kk-scene-package/2`: retain `scene-package.json`, terrain, and props; include a verified, complete display-only `.skin/3` as the default robot under `robot/`, bound to a trusted profile SHA. Spawn must be declared or explicitly supplied by the exporter. Legacy `/1` and `.scene.zip` are read-compatible only. Attach the scene; do not concatenate XML in a way that overrides the robot's global physics parameters. `spawn.yaw` remains in degrees, never radians. Compile the default complete robot and check its spawn in each new map; manifest fields alone do not prove that a robot is visible.
- Package data must be self-contained. Reject path traversal, undeclared assets, hash mismatches, unknown required capabilities, and executable extensions. A failed check must stop release/replacement and preserve the previous session and files.
- Protocol changes must update schemas, the Python reference validator, round-trip and rejection tests, docs, and release notes. New mandatory semantics require a protocol version bump; do not keep `/1` if old readers would silently ignore a new requirement. Cross-platform support applies only to consumers that implement the protocol and required capabilities.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KingKongRobotics/kingkong-design](https://github.com/KingKongRobotics/kingkong-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
