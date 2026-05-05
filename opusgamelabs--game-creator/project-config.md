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
  game-assets/SKILL.md     # Pixel art sprites (code-only, no external files)
  game-designer/SKILL.md   # Visual polish (gradients, particles, juice, transitions)
  game-audio/SKILL.md      # Procedural audio (Web Audio API BGM + SFX)
  game-qa/SKILL.md         # Playwright testing (gameplay, visual, perf)
  game-architecture/SKILL.md  # Reference architecture patterns
  game-deploy/SKILL.md     # Deployment (here.now default, GitHub Pages, Vercel, etc.)
  use-template/SKILL.md    # Clone a gallery template as a starting point
  playdotfun/SKILL.md      # Play.fun monetization (git submodule → submodules/playdotfun)
  promo-video/SKILL.md     # Autonomous 50 FPS gameplay recording (Playwright + FFmpeg)
  make-game/SKILL.md       # Full pipeline: scaffold → assets → design → promo video → audio → deploy (here.now) → monetize (QA at every step)
  improve-game/SKILL.md    # Holistic audit + implement highest-impact improvements
  design-game/SKILL.md     # Visual design audit + improvements
  add-feature/SKILL.md     # Add feature following patterns
  add-assets/SKILL.md      # Replace shapes with pixel art sprites
  game-3d-assets/SKILL.md  # 3D model pipeline (GLB download, AssetLoader, animated characters)
  meshyai/SKILL.md         # Meshy AI — generate 3D models from text/images, auto-rig, animate
  add-3d-assets/SKILL.md   # Replace 3D primitives with real GLB models
  add-audio/SKILL.md       # Add procedural audio (Web Audio API)
  record-promo/SKILL.md    # Record autonomous promo video (standalone command)
  monetize-game/SKILL.md   # Play.fun monetization (register, SDK, redeploy)
  qa-game/SKILL.md         # Add Playwright QA tests
  review-game/SKILL.md     # Code review for architecture + best practices
templates/
  phaser-2d/               # Runnable 2D starter project (Phaser 3)
  threejs-3d/              # Runnable 3D starter project (Three.js)
scripts/
  iterate-client.js        # Standalone Playwright iterate loop (action → screenshot → state → errors)
  example-actions.json     # Example action payloads for iterate-client.js
  find-3d-asset.mjs        # Search & download GLB models (Sketchfab, Poly Haven, Poly.pizza)
  meshy-generate.mjs       # Generate 3D models with Meshy AI (text-to-3d, image-to-3d, rig, animate)
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
    Dockerfile               # Railway container
    package.json             # Express, pg, cors
submodules/
  playdotfun/              # Git submodule: github.com/OpusGameLabs/skills
agents/
  game-creator.md          # Autonomous game creation pipeline with build/visual gates
  game-deploy.md           # Deployment automation (preloads game-deploy skill)
  game-qa-runner.md        # Test runner + autofix (preloads game-qa, game-architecture)
  game-reviewer.md         # Code review agent (preloads game-architecture)
examples/
  flappy-bird/             # Complete example game (see below)
  nick-land-dodger/        # Dodge game with photo-composite character + promo video
  3d-asset-test/           # 3D asset pipeline demo (animated characters, model loading, OrbitControls)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpusGameLabs/game-creator](https://github.com/OpusGameLabs/game-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
