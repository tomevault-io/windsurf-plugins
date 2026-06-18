---
trigger: always_on
description: Build, extend, and deploy Decentraland SDK7 scenes. Contains agent behavioral guidelines, the composite-first rule, and an index of all topic skills with reference files. Install with a single command — no flags needed.
---


# Decentraland SDK7 Scene Development

> **Runtime constraint:** Decentraland runs in a QuickJS sandbox. No Node.js APIs (`fs`, `http`, `path`, `process`). Use `executeTask()` + `fetch()` for async work.

## Agent Behavioral Guidelines

Before taking any significant action, check whether it falls into one of the three categories below and **confirm with the user first**.

**How to ask:** Phrase the question in plain, non-technical language that describes _what will happen to the scene_, not the underlying command.

> Good: _"Should I download this tree model into your scene assets?"_
> Bad: _"Run `curl https://… -o assets/Models/tree.glb`?"_

### 1. Changing parcel count or layout

Any modification to `scene.parcels` in scene.json changes the scene's coordinate space. Entities near the current boundary may end up outside (invisible) or inside the wrong parcel. The user may also have a deployment slot in mind and parcel count needs to match it. Describe the change and its effect before acting:

> "To fit the scene you described, I'd need to expand from 1 parcel (16×16 m) to a 2×1 layout (32×16 m). This changes the coordinate bounds for every entity. Should I go ahead?"

### 2. Fetching assets from external sources

Downloading any file not already in the project — 3D models (.glb), images, audio, video. The user may have their own assets in mind, may not want new files added, or may be targeting a specific visual style. Confirm before downloading:

> "I'd like to download a [description] model from [source] and add it to your scene. Should I go ahead?"

For streaming references (`AudioStream`, `VideoPlayer`): these don't download files but do add an external URL dependency. Confirm if the URL wasn't provided by the user:

> "I'd set up a video stream from [source]. Is that the one you want to use?"

### 3. Adding an authoritative server

Introduces `isServer()`, `registerMessages()`, `Storage`, `EnvVar`, or switches to `@dcl/sdk@auth-server`. This feature requires switching to an alternative SDK branch (`@dcl/sdk@auth-server`). Many users who want "multiplayer" only need the simpler `multiplayer-sync` skill (no server). Confirm before implementing:

> "To handle multiplayer this way I'd need to add an Authoritative Server — that requires switching to the `@dcl/sdk@auth-server` SDK branch instead of the standard one. Is that what you're after, or would simpler peer-to-peer sync work for your use case?"

### General principle

These aren't things the agent should refuse to do — they're things it should communicate about before doing them. If the user confirms, proceed confidently. The goal is transparency, not gatekeeping.

---

## CRITICAL RULE — Composite-first scene authoring

All static entities (models, lights, spawn points) MUST be defined in `assets/scene/main.composite`, NOT created in TypeScript via `engine.addEntity()`.

TypeScript (`src/index.ts`) is ONLY for:
- Dynamic behavior (systems, event handlers, state changes)
- Referencing composite entities via `getEntityOrNullByName('name')` or `getEntitiesByTag('tag')`
- Entities that are truly runtime-only (spawned/despawned during gameplay)

---

## Individual Skills

This skill is the entry point. The detailed implementation guidance lives in individual topic skills, each installable separately. Install specific ones or use `--skill '*'` for all.

### Scene Setup & Configuration
**Skill: `create-scene`** — Scaffolding, `scene.json` schema, multi-parcel layouts, composite vs TypeScript entity rules.

### 3D Models
**Skill: `add-3d-models`** — Loading `.glb`/`.gltf` with `GltfContainer`, positioning, colliders, and browsing the free asset catalogs (8,800+ models).

### Animations & Tweens
**Skill: `animations-tweens`** — GLTF animation clips with `Animator`, programmatic `Tween` and `TweenSequence`, easing functions.

### Materials & Rendering
**Skill: `advanced-rendering`** — PBR materials, `TextShape`, `Billboard`, `VisibilityComponent`, texture modes.

### Lighting & Environment
**Skill: `lighting-environment`** — Point/spot lights, shadows, `SkyboxTime` (day/night cycle), emissive materials.

### Particle Systems
**Skill: `particle-system`** — `ParticleSystem` component for fire, smoke, sparks, snow, rain, magic, fireworks. Emitter shapes (Point/Sphere/Cone/Box), continuous rate vs Burst emission, gravity, sprite-sheet animation, blend modes.

### Click & Proximity Interactivity
**Skill: `add-interactivity`** — `pointerEventsSystem`, trigger areas, raycasting. For polling-based input see `advanced-input`.

### Advanced Input & Movement Control
**Skill: `advanced-input`** — `inputSystem` polling, WASD-controlled entities, `InputModifier`, `PointerLock`, `PrimaryPointerInfo`.

### Player & Avatar
**Skill: `player-avatar`** — Player position/profile, emotes, wearables, `AvatarAttach`, `AvatarModifierArea`.

### NPCs
**Skill: `npcs`** — `AvatarShape` NPCs and the NPC Toolkit library for GLB-based NPCs with dialogue and state machines.

### Player Physics
**Skill: `player-physics`** — Impulse forces, knockback, repulsion fields.

### Camera

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [decentraland/sdk-skills](https://github.com/decentraland/sdk-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
