---
trigger: always_on
description: This repository contains the official documentation for the **Dojo** ecosystem - a comprehensive toolchain for building provable games and autonomous worlds on Starknet. Dojo provides a complete framework for developing fully on-chain games using Cairo smart contracts with Entity Component System (ECS) architecture.
---

# CLAUDE.md - Dojo Book Documentation Project

## Project Overview

This repository contains the official documentation for the **Dojo** ecosystem - a comprehensive toolchain for building provable games and autonomous worlds on Starknet. Dojo provides a complete framework for developing fully on-chain games using Cairo smart contracts with Entity Component System (ECS) architecture.

### Project Type

- **Type**: Technical documentation website built with Vocs (Vue-based documentation framework)
- **Primary Technology**: TypeScript/JavaScript with React components
- **Content Format**: Markdown/MDX files with code examples
- **Deployment**: Vercel hosting with automatic builds
- **Architecture**: Doc source in docs/, with reference material kept in src/ as Git submodules

### Content Management

- **Markdown/MDX**: Documentation content with JSX components
- **Git Submodules**: Source code references from multiple repositories
- **Structured Navigation**: Hierarchical routing with collapsible sections

## Build & Development Commands

### Development Workflow

```bash
# Start development server (http://localhost:5173/)
pnpm run dev

# Build for production
pnpm run build
```

### Content Management

```bash
# Update all submodules to latest versions
git submodule update --remote

# Check submodule status
git submodule status
```

## High-Level Architecture

### Documentation Structure

```
~/book/
├── docs/                  # Documentation content
│   ├── components/        # React components for docs
│   ├── pages/             # MDX content files
│   │   ├── architecture/  # Technical architecture (execution, rollups)
│   │   ├── blog/          # Blog posts and articles
│   │   ├── client/        # SDK documentation for multiple platforms
│   │   │   └── sdk/       # Platform-specific SDKs
│   │   │       ├── bevy/  # Bevy engine integration // REFERENCE: src/sdks/dojo.bevy
│   │   │       ├── c/     # C language bindings // REFERENCE: src/sdks/dojo.c
│   │   │       ├── discord/ # Discord integration
│   │   │       ├── godot/ # Godot engine integration
│   │   │       ├── javascript/ # JavaScript SDK // REFERENCE: src/sdks/dojo.js
│   │   │       ├── rust/  # Rust SDK
│   │   │       ├── telegram/ # Telegram integration
│   │   │       ├── unity/ # Unity integration // REFERENCE: src/sdks/dojo.unity
│   │   │       └── unreal/ # Unreal Engine integration // REFERENCE: src/sdks/dojo.unreal
│   │   ├── community/     # Community resources
│   │   ├── framework/     # Core Dojo framework documentation // REFERENCE: src/framework/dojo && src/starters && src/games
│   │   │   ├── authorization/ # Authorization system
│   │   │   ├── config/    # Configuration management
│   │   │   ├── models/    # Model definitions and API
│   │   │   ├── systems/   # Systems documentation
│   │   │   ├── testing/   # Testing framework and cheat codes
│   │   │   └── world/     # World contract (API, events, metadata, permissions)
│   │   ├── getting-started/ # Onboarding and setup guides
│   │   ├── libraries/     # Ecosystem libraries
│   │   │   ├── alexandria/ # Alexandria library collection
│   │   │   └── origami/   # Cairo utilities library // REFERENCE: src/libraries/origami
│   │   ├── misc/          # Miscellaneous content
│   │   ├── theory/        # Conceptual topics (autonomous worlds)
│   │   ├── toolchain/     # Tool-specific documentation
│   │   │   ├── katana/    # Starknet sequencer // REFERENCE: src/toolchain/katana
│   │   │   ├── saya/      # Proving system // REFERENCE: src/toolchain/saya
│   │   │   ├── slot/      # Deployment commands
│   │   │   ├── sozo/      # Project and world commands // REFERENCE: src/framework/dojo
│   │   │   └── torii/     # Indexing engine // REFERENCE: src/toolchain/torii
│   │   └── tutorials/     # Step-by-step learning guides // REFERENCE: src/starters && src/games
│   └── public/            # Static assets
├── src/                   # Git submodules (reference repositories)
├── spec/                  # Documentation standards
└── configuration files    # Build, linting, deployment
```

### Key Directories

#### `/docs/pages/` - Content Organization

- **Framework**: Core Dojo concepts (World, Models, Systems) -- theoretically focused
- **Toolchain**: Tool-specific documentation (Katana, Torii, Sozo, Saya) -- theoretically focused
- **Tutorials**: Step-by-step learning guides -- practically focused
- **Client/SDK**: Integration guides for various platforms -- practically focused
- **Architecture**: Scaling and technical deep-dives -- theoretically focused
- **Libraries**: Ecosystem libraries (Origami, Alexandria) -- practically focused

#### `/src/` - Source Code Submodules

- **Framework**: Core Dojo framework implementation (Rust/Cairo)
- **Games**: Example game implementations showcasing Dojo capabilities
- **Libraries**: Ecosystem libraries like Origami (Cairo utilities)
- **SDKs**: Multi-platform integrations (JavaScript, Unity, Unreal, Bevy, C/C++)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dojoengine/book](https://github.com/dojoengine/book) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
