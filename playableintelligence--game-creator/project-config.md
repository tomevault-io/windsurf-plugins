---
trigger: always_on
description: This is **game-creator**, the game studio for the agent internet. It provides skills and agents for scaffolding, designing, deploying, and monetizing 2D (Phaser 3) and 3D (Three.js) browser games. QA (build, runtime, visual review, autofix) runs at every step. Monetize with [Play.fun](https://play.fun) (OpenGameProtocol). Works with **40+ AI coding agents** (via `npx skills add`). Share your play.fun URL on [Moltbook](https://www.moltbook.com/).
---

# CLAUDE.md

## Project Overview

This is **game-creator**, the game studio for the agent internet. It provides skills and agents for scaffolding, designing, deploying, and monetizing 2D (Phaser 3) and 3D (Three.js) browser games. QA (build, runtime, visual review, autofix) runs at every step. Monetize with [Play.fun](https://play.fun) (OpenGameProtocol). Works with **40+ AI coding agents** (via `npx skills add`). Share your play.fun URL on [Moltbook](https://www.moltbook.com/).

## Repository Structure

```
.claude-plugin/
  plugin.json              # Plugin manifest (name, version, author)
  marketplace.json         # Marketplace metadata (owner: OpusGameLabs)
settings.json              # Default settings (activates game-creator agent)
skills/
  phaser/SKILL.md          # 2D game patterns (Phaser 3, scene-based, multi-file)
  threejs-game/SKILL.md    # 3D game patterns (Three.js, event-driven)
  threejs-perf/SKILL.md    # Three.js performance patterns with measured evidence (instancing static + moving)
  game-assets/SKILL.md     # Pixel art sprites (code-only, no external files)
  game-designer/SKILL.md   # Visual polish (gradients, particles, juice, transitions)
  game-audio/SKILL.md      # Procedural audio (Web Audio API BGM + SFX)
  game-qa/SKILL.md         # Playwright testing (gameplay, visual, perf)
  game-architecture/SKILL.md  # Reference architecture patterns
  game-deploy/SKILL.md     # Deployment (here.now default, GitHub Pages, Vercel, etc.)
  use-template/SKILL.md    # Clone a gallery template as a starting point
  playdotfun/SKILL.md      # Play.fun monetization (git submodule → submodules/playdotfun)
  promo-video/SKILL.md     # Autonomous 50 FPS gameplay recording (Playwright + FFmpeg)
  make-game/SKILL.md       # Multi-session game-dev pipeline: idea → scaffold → development phases with milestones, ADRs, docs/STATE.md
  viral-game/SKILL.md      # One-shot viral pipeline: tweet/story/concept → scaffold → assets → design → promo video → audio → deploy (here.now) → monetize (QA every step, ~10 min)
  improve-game/SKILL.md    # Holistic audit + implement highest-impact improvements
  design-game/SKILL.md     # Visual design audit + improvements
  add-feature/SKILL.md     # Add feature following patterns
  add-assets/SKILL.md      # Replace shapes with pixel art sprites
  retrodiffusion/SKILL.md  # Retro Diffusion — true pixel art via paid API (sprites, tilesets, animations)
  game-3d-assets/SKILL.md  # 3D model pipeline (GLB download, AssetLoader, animated characters)
  meshyai/SKILL.md         # Meshy AI — generate 3D models from text/images, auto-rig, animate
  add-3d-assets/SKILL.md   # Replace 3D primitives with real GLB models
  add-audio/SKILL.md       # Add procedural audio (Web Audio API)
  record-promo/SKILL.md    # Record autonomous promo video (standalone command)
  monetize-game/SKILL.md   # Play.fun monetization (register, SDK, redeploy)
  scaffold-gateables/SKILL.md  # Add monetization-agnostic gateable features (isEntitled hooks + skin picker, continue, etc.)
  add-multiplayer/SKILL.md # Add real-time or turn-based multiplayer via PartyKit (Cloudflare Durable Objects)
  qa-game/SKILL.md         # Add Playwright QA tests
  sub-games/SKILL.md       # Sub.games community platform for finding players and supporters
  review-game/SKILL.md     # Code review for architecture + best practices
  unity-mcp/SKILL.md       # Unity Editor automation via MCP (git submodule → submodules/unity-mcp, beta branch)
templates/
  phaser-2d/               # Runnable 2D starter project (Phaser 3)
  threejs-3d/              # Runnable 3D starter project (Three.js)
scripts/
  iterate-client.js        # Standalone Playwright iterate loop (action → screenshot → state → errors)
  example-actions.json     # Example action payloads for iterate-client.js
  find-3d-asset.mjs        # Search & download GLB models (Sketchfab, Poly Haven, Poly.pizza)
  meshy-generate.mjs       # Generate 3D models with Meshy AI (text-to-3d, image-to-3d, rig, animate)
  retrodiffusion-generate.mjs  # Generate pixel art with Retro Diffusion (generate, img2img, animate, tileset, edit, balance)
assets/
  characters/              # 2D South Park-style spritesheets (photo-composite)
    manifest.json
    characters/
  3d-characters/           # Animated GLB characters with clip maps
    manifest.json
    models/                # Soldier.glb, Xbot.glb, RobotExpressive.glb, Fox.glb
site/
  manifest.json              # Source of truth: metadata for all templates
  build.js                   # Unified build → _site/index.html + _site/gallery/index.html
  capture-screenshots.js     # Playwright: auto-capture thumbnails for each game
  thumbnails/                # 400x225 PNGs (committed to git)
  telemetry/
    server.js                # Express telemetry API (ingestion + stats)
    schema.sql               # PostgreSQL schema

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PlayableIntelligence/game-creator](https://github.com/PlayableIntelligence/game-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
