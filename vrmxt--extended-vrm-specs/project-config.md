---
trigger: always_on
description: How to author Extended VRM specs, ADRs, example notes, and schema docs
---


# Specs documentation

This repository holds Extended VRM specifications and related design decisions.
Prefer clarity for implementers over essay voice.

## Normative writing

- Separate **requirements** from **rationale** and **examples**. Label examples as non-normative.
- Prefer concrete field names, types, units, defaults, and version bounds over adjectives.
- Use consistent requirement strength (MUST / SHOULD / MAY, or an equivalent house convention).
  Do not mix soft marketing language into normative sections.
- State compatibility explicitly: which VRM/glTF versions, packages, or tools a change targets.
- When a requirement is undecided, write `TBD` or `[ADD: …]` — do not invent a decision.

## Audience

Separate **implementer / product** prose from **repo maintainer / agent** process.

| Audience | Belongs in | Examples |
|----------|------------|----------|
| Creators using shipping tools | `tutorials/` | Install, author, export, Warudo getting started / patch |
| Spec readers, tool authors, consumers | `specs/`, `decisions/`, `implementations/`, `references/` | Field rules, catalog schema, vendor paths, UI policy (`common: true`) |
| Readers of a loaded file | `examples/` | Images, clipped JSON, stack mermaid (Warudo MToonXT + lilToon) |
| Specs maintainers and Cursor agents | `.cursor/rules/`, `.cursor/skills/`, optional `references/**/maintaining-*.md` | Regen scripts, same-PR sync habits, deslop triggers, skill workflows |

- Do **not** put `.cursor/**` paths in normative specs or in family/reference notes aimed at
  implementers. Point those notes at a `maintaining-*.md` page (or the skill) instead.
- Do **not** move product authoring policy (how tools use catalogs, morph layers, etc.) into
  Cursor rules — tools still need that in the vault.
- Agent-only steps that already live in a skill (e.g. `unity-shader-catalog`) stay in the
  skill; the maintaining page is the human-facing index when one is needed.

Materials-override catalog regen: [Maintaining catalogs](../../references/catalogs/maintaining-catalogs.md)
and skill `unity-shader-catalog`.

When adding or changing a shipping **editor** host: update the capability matrix in
[VRMXT Editor](../../implementations/vrmxt-editor.md) and the Architecture authoring
host table. Prefer the host profile for per-host detail; keep this matrix aligned.

## Structure

- Format authored notes for Obsidian using `obsidian-markdown.mdc`, including YAML
  properties and controlled tags.
- One concern per document when practical. Link related specs instead of duplicating rules.
- Keep headings scannable: purpose, scope, normative text, data model / schema, examples,
  compatibility, open questions.
- Tutorials (`tutorials/`): second person; what you will have at the end; what you
  need; numbered UI steps. Authoring pages stop at export / re-import in that host.
  Do not define success as one consumer (Warudo, UniVRMXT, …). Consumer-specific
  checks belong on that consumer's tutorial. Spec field rules go in Related.
  Deslop these as tutorial / how-to, not as specifications.
- Prefer tables for fields, enums, and matrices. Prefer diagrams only when they clarify
  relationships that prose would bury.
- When a diagram is needed, use a fenced **Mermaid** block (` ```mermaid `). Do not use
  ASCII art boxes, arrows, or trees for flow, sequence, state, or relationship diagrams.
  Keep node/edge labels short and stable; prefer `flowchart`, `sequenceDiagram`,
  `stateDiagram-v2`, or `classDiagram` over decorative layouts.
- Preserve stable anchors and filenames once published; renames need intentional migration.

## File placement

- Put family-wide normative contracts that are not glTF extensions in `specs/core/`.
- Put reusable normative schema fragments with no independent `extensionsUsed` identity
  in `specs/fragments/`.
- Put concrete glTF extension specs in `specs/extensions/<domain>/`. Current domains are
  `animation`, `deformation`, `materials`, `physics`, and `vfx`.
- Do not place a concrete extension spec directly under `specs/`.
- Folder names are editorial taxonomy. They MUST NOT add a segment to the serialized
  extension name. The path stem maps by replacing hyphens with underscores:
  `specs/extensions/vfx/vrmxt-sprite-particle.md` defines `VRMXT_sprite_particle`.
  A folder `vrmxt-materials-mtoonxt/` is the same stem (multi-page spec).
- Serialized glTF extension names authored here MUST use `VRMXT_*`. Do not invent
  `VRMC_*` names. `VRMC_` is VRM Consortium only (`VRMC_vrm`, `VRMC_materials_mtoon`,
  `VRMC_springBone`, `VRMC_springBone_extended_collider`, …).
- Sitting next to a stock object does not change the prefix. Pairing is `…xt` (family
  fork) and `_override` (third-party replace). MToon:
  `VRMXT_materials_mtoonxt` / `VRMXT_materials_override`. Spring:
  `VRMXT_springBonext` / `VRMXT_springBone_override`. Path exception: camelCase
  `Bone` in `vrmxt-springbonext/` → `VRMXT_springBonext`. Details:
  `architecture.md` Naming.
- Inner JSON keys stay unprefixed camelCase (`specVersion`, `stencil`, `faceSdf`).
  Do not prefix properties `VRMC_` or `VRMXT_`.
- Code types SHOULD use `Vrmxt*` / `vrmxt_*`. Stock UniVRM/MToon10 hlsl includes keep
  `vrmc_materials_mtoon_*.hlsl`. ShaderLab `VRMXT/...` is not a glTF key.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vrmxt/Extended-VRM-Specs](https://github.com/vrmxt/Extended-VRM-Specs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
