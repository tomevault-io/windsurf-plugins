---
trigger: always_on
description: Avatune is a monorepo combining ML-powered avatar analysis with browser-based avatar rendering. It consists of:
---

## Project Overview

Avatune is a monorepo combining ML-powered avatar analysis with browser-based avatar rendering. It consists of:

1. **Python ML Training** - TensorFlow/Keras models trained on CelebA and FairFace datasets, converted to TensorFlow.js
2. **TypeScript Packages** - Browser-compatible ML predictor packages using TensorFlow.js
3. **Avatar Rendering** - Avatar generation from analysis results

The workflow: Python trains models → exports to TFJS → TypeScript packages load models → browser inference.

## Architecture

### Monorepo Structure

```
avatune/
├── apps/                                    # Applications
│   ├── website/                             # Documentation website (Astro)
│   ├── cloudflare-worker/                   # Cloudflare Worker API
│   ├── RNStorybook/                         # React Native Storybook
│   ├── react-storybook/                     # React Storybook
│   ├── svelte-storybook/                    # Svelte Storybook
│   ├── vue-storybook/                       # Vue Storybook
│   ├── vanilla-storybook/                   # Vanilla JS Storybook
│   ├── predictor-storybook/                 # ML Predictor demos
│   ├── studio/                              # Theme creation studio
│   └── storybook-root/                      # Root Storybook aggregator
├── packages/
│   ├── assets/                              # SVG assets per theme
│   │   ├── ashley-seo-assets/
│   │   ├── fatin-verse-assets/
│   │   ├── kyute-assets/
│   │   ├── micah-assets/
│   │   ├── miniavs-assets/
│   │   ├── nevmstas-assets/
│   │   ├── pacovqzz-assets/
│   │   ├── pawel-olek-assets/
│   │   └── yanliu-assets/
│   ├── themes/                              # Theme configurations
│   │   ├── ashley-seo-theme/
│   │   ├── fatin-verse-theme/
│   │   ├── kyute-theme/
│   │   ├── micah-theme/
│   │   ├── miniavs-theme/
│   │   ├── nevmstas-theme/
│   │   ├── pacovqzz-theme/
│   │   ├── pawel-olek-man-theme/
│   │   ├── pawel-olek-woman-theme/
│   │   └── yanliu-theme/
│   ├── renderers/                           # Platform-specific renderers
│   │   ├── react/
│   │   ├── react-native/
│   │   ├── solidjs/
│   │   ├── svelte/
│   │   ├── vue/
│   │   └── vanilla/
│   ├── predictors/                          # ML prediction packages
│   │   ├── face-detector/
│   │   ├── hair-color-predictor/
│   │   ├── hair-length-predictor/
│   │   └── skin-tone-predictor/
│   ├── core/                                # Shared core packages
│   │   ├── types/                           # TypeScript types
│   │   ├── utils/                           # Shared utilities
│   │   ├── theme-builder/                   # Theme builder API
│   │   ├── api-client/                      # API client
│   │   └── typescript-config/               # Shared TS configs
│   └── rsbuild-plugins/                     # Build plugins
│       ├── rsbuild-plugin-copy-tfjs-model/
│       ├── rsbuild-plugin-raw-svg/
│       ├── rsbuild-plugin-svg-to-solid/
│       ├── rsbuild-plugin-svg-to-svelte/
│       └── rsbuild-plugin-svg-to-vue/
├── scripts/                                 # Build/generation scripts
│   ├── generate-assets.ts                   # Generate asset entrypoints
│   ├── generate-theme.ts                    # Scaffold new themes
│   ├── generate-stories.ts                  # Generate Storybook stories
│   ├── generate-assets-readme.ts            # Generate asset READMEs
│   ├── generate-assets-theme-readme.ts      # Generate theme READMEs
│   ├── generate-themes-mdx.ts               # Generate theme docs
│   ├── generate-root-readme.ts              # Generate root README
│   └── shared.ts                            # Shared script utilities
└── python/                                  # ML training pipeline
    ├── notebooks/                           # Marimo notebooks
    │   ├── hair_color/
    │   ├── hair_length/
    │   └── skin_tone/
    ├── data/                                # Training datasets (gitignored)
    └── models/                              # Trained models + TFJS exports
```

### Key Technologies

- **Turborepo** - Monorepo orchestration with caching
- **Bun** - Package manager (specified in package.json)
- **Biome** - Linting and formatting (replaces ESLint/Prettier)
- **Rslib** - Library bundler for packages (dual ESM/CJS)
- **Rsbuild** - App bundler (Rspack-based, faster than Webpack)
- **Storybook** - Component demos
- **TensorFlow.js** - Browser-based ML inference
- **uv** - Python package manager (fast pip alternative)
- **Marimo** - Interactive Python notebooks
- **Astro** - Documentation website

## Common Commands

### Root Level

```bash
bun install              # Install dependencies
bun run build            # Build all packages and apps
bun dev                  # Dev mode (all workspaces with watch)
bun storybook            # Run all storybooks
bun lint                 # Lint all workspaces
bun format               # Format all code
bun run check-types      # Type checking
```

### Scripts

```bash
# Generate asset entrypoints from SVG files
bun scripts/generate-assets.ts <assets-package-name>
# Example: bun scripts/generate-assets.ts kyute-assets

# Scaffold a new theme from assets package
bun scripts/generate-theme.ts <theme-name>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [avatune/avatune](https://github.com/avatune/avatune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
