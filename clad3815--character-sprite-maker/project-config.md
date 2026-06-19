---
trigger: always_on
description: Create, configure, repair, validate, preview, and package arbitrary 2D game character spritesheets from text concepts, style settings, animation lists, screenshots, generated images, or visual references. Use when a user wants a configurable character generator for games, e.g. platformer, Mario-like original mascot, RPG, fighting-game, metroidvania, top-down, isometric, idle/walk/run/jump/attack animations, custom frame counts, layout guides, transparent conversion, contact sheets, GIF previews,
---


# Character Sprite Maker

## Overview

Create a configurable animated 2D character spritesheet from a concept, one or more reference images, or both. This skill generalizes the hatch-pet pipeline: it keeps the reliable pieces (prompt planning, references, chroma-key conversion, frame extraction, atlas composition, validation, contact sheet, previews, packaging) but removes pet-specific constraints such as a fixed 8x9 atlas, fixed animation rows, and Codex pet packaging.

The user may specify:

- character type or concept, e.g. "original plumber-like platformer hero", "top-down wizard", "fighting game boss", "robot NPC", "Mario-like 2D platformer character";
- visual style preset and extra style notes;
- animation preset or a custom animation list such as `idle`, `walk-right`, `walk-left`, `jump`, `attack`, `hurt`, etc.;
- frame counts per animation;
- cell size, atlas columns, FPS, view/camera, references, and package location.

If a field is missing, infer a practical default and continue. Only ask the user when the missing choice is truly blocking.

## Generation Delegation

Use `$imagegen` for all normal visual generation.

Before generating base art, animation strips, or repairs, load and follow the installed image generation skill when available:

```text
${CODEX_HOME:-$HOME/.codex}/skills/.system/imagegen/SKILL.md
```

Do not call the Image API directly for the normal path. Let `$imagegen` choose its own built-in-first path and CLI fallback rules. If `$imagegen` says a fallback requires confirmation, ask the user before continuing.

Use this skill's scripts only for deterministic work: preparing prompts/manifests, copying references, creating layout guides, ingesting selected `$imagegen` outputs, extracting frames, removing chroma-key backgrounds, validating rows, composing the final atlas, creating QA media, and packaging.

Hard boundary: do not create, draw, tile, warp, or synthesize character visuals with local Python/Pillow scripts, SVG, canvas, HTML/CSS, or code-native art as a substitute for `$imagegen`. Local scripts may process already-generated visual outputs only.

## Copyright / Style Handling

The user can request a genre or broad style, e.g. "style Mario", "Sonic-like platformer energy", "Zelda-like top-down RPG". Convert that into an original character and general art direction. Do not generate an exact copy of copyrighted characters, logos, trademarked costumes, or protected game assets unless the user provides rights/ownership context and the request is allowed.

For example, "personnage 2D style Mario" should become an "original, bright, family-friendly mascot-platformer sprite with rounded readable shapes and bold colors", not Mario himself.

## Input Model

Use `scripts/prepare_character_run.py`. It supports direct CLI flags or a JSON config file.

Important flags:

```bash
python "$SKILL_DIR/scripts/prepare_character_run.py" \
  --character-name "<Name>" \
  --character-notes "<one or two sentences describing the character>" \
  --style-preset pixel-platformer \
  --style-notes "<extra art direction>" \
  --view "side-view" \
  --animation idle:6:"neutral idle loop" \
  --animation walk-right:8:"rightward walk cycle" \
  --animation walk-left:8:"leftward walk cycle" \
  --cell-width 128 \
  --cell-height 128 \
  --reference /absolute/path/to/reference.png \
  --output-dir /absolute/path/to/run \
  --force
```

All arguments are optional except those needed to express the user's constraints. If no animations are supplied, use the `platformer` preset by default. If the user mentions Mario-like, platformer, top-down RPG, fighting game, or pet-compatible, use the matching preset when appropriate.

Available style presets:

- `pixel-platformer`
- `mario-like`
- `topdown-rpg`
- `metroidvania`
- `fighting-game`
- `cartoon-hd`
- `iso-rpg`

Available animation presets:

- `platformer`
- `mario-like`
- `topdown-rpg`
- `fighting`
- `pet-compatible`

Animation specs support:

```text
name
name:frames
name:frames:action description
name=frames:action description
```

Examples:

```bash
--animation idle:6:"breathing and blink loop"
--animation walk-right:8:"rightward platformer walk cycle"
--animation attack:6:"short punch attack"
```

## Visible Progress Checklist

For a normal character run, keep a visible checklist for the user:

1. Configuring `<Character>`.
2. Creating `<Character>`'s base look.
3. Generating `<Character>`'s animation strips.
4. Converting and validating `<Character>`.
5. Packaging `<Character>`.

Only mark a step complete when the real file, image, or decision exists.

## Default Workflow

1. Prepare the run folder and imagegen job manifest:

```bash
SKILL_DIR="${CODEX_HOME:-$HOME/.codex}/skills/character-sprite-maker"
python "$SKILL_DIR/scripts/prepare_character_run.py" \
  --character-name "<Name>" \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Clad3815/character-sprite-maker](https://github.com/Clad3815/character-sprite-maker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
