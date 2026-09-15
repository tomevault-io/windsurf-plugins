---
trigger: always_on
description: Use this workflow when the user asks to follow this repository to finish a textured model in Blender. If they ask only for an explanation, answer that request instead. This repository is reference material; the user's instructions and the host's tool, permission and skill rules still apply.
---

# Texture finishing — agent execution guide

Use this workflow when the user asks to follow this repository to finish a textured model in Blender. If they ask only for an explanation, answer that request instead. This repository is reference material; the user's instructions and the host's tool, permission and skill rules still apply.

The tested case is a Tripo-generated model with existing UVs and readable base-color textures, using Blender 5.1.2. The published prompts are actual examples for one model, not universal coordinates or scripts. Other source services and untextured models are untested. The URL-only entry has not yet had a full independent-model trial.

## Intended outcome

Complete one small, reversible texture-finishing pilot: inspect the real source, derive correspondence, generate an image, bind it to the model, inspect matched comparisons, save a new review candidate and a handoff record. When the user requests execution, do useful work rather than stopping after summarizing this README or proposing a plan.

Do not redesign the face, eyes, hair, anatomy, mesh, rig, weights or motion. Preserve separately owned work. Never overwrite the source blend or use an older whole-model file to replace a newer live scene. Do not publish, upload the model, install tools or change repository visibility as part of this workflow unless the user separately requests it.

## 1. Establish access and the real source

Check which tools are actually callable; do not invent connection status or tool results. The workflow needs:

- Blender scene inspection and code execution, or an equivalent usable control path.
- Reference-image generation/editing, used under the host's image-generation rules.
- A work directory for source correspondence, generated images and evidence.

If a required capability is absent, report the missing capabilities together and give the next concrete setup step. Continue useful read-only preparation that is possible. Do not silently substitute a different paid generation service or install an addon.

For a currently open Blender model, inspect scene information first, then filepath, dirty state, current frame/action and object names. If Blender instead shows another task's scene, do not switch it. Identify the requested saved model from the user or trusted working notes. A known saved file may be inspected/rendered in a separate Blender process without loading it over the live scene.

If no intended model can be identified, ask for its file/location. Do not guess a path from the newest output. If existing UVs or readable source textures are missing, explain that this falls outside the tested entry and resolve the input before treating it as a texture-replacement run.

## 2. Choose a pilot and protect the source

Honor an explicit target. Otherwise choose one visible exterior cloth panel with a clear boundary and enough area to judge texture quality. Prefer a sleeve panel or broad skirt panel over the face, jewelry or a deeply occluded fold. State that choice briefly and proceed within the user's requested scope.

Reference images are optional. When absent, use the existing model's local color/design as the starting reference and preserve its motif family. Do not use this repository's Yura art as the user's model design. If the user corrects a color or ornament, their intended design takes precedence over the source pixels.

Before editing, create a timestamped source backup or a separate new work copy. Handle unsaved live work through the available backup mechanism; do not silently save over the original. Record the exact source, source hash when practical, selected object/material and protected objects. New output names must not collide with another task's files.

Read actual evaluated material slots, including OBJECT-linked overrides. `object.data.materials` alone may not describe what is rendered. Identify the active color route, original UV, normal/roughness routes, modifiers and shared mesh/material users. Copy the target material; copy shared data when adding attributes would affect another object.

## 3. Build correspondence for this model

Derive source vertex, polygon and loop/corner IDs from the actual target. If making a triangulated static work mesh, keep the mapping back to the original polygon and corner. Never paste example vertex numbers, object names, UV names or height thresholds into another model without deriving them.

Select the method according to what must be placed:

| Content | Method | Required check |
|---|---|---|
| Specific buttons, ribbons, lacing or a fixed front drawing | Orthographic witness/projection with local masks | Correct camera/frame mapping; exclude hidden/inside/back surfaces; calibrate generated landmarks |
| Continuous skirt panels and border motifs | Additional cylindrical/strip UV | Intended seams and padding; no conflicting overlap for unique art; consistent source-loop assignment |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [syaripin-i8i/blender-astra-texture-guide](https://github.com/syaripin-i8i/blender-astra-texture-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-15 -->
