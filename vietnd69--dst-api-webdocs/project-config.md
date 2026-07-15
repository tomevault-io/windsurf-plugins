---
trigger: always_on
description: Manages all aspects of character behavior, customization, and progression.
---

# DST Core Systems Index Templates

This rule provides specific templates for different types of index.md files within the core-systems directory structure. Each template is optimized for its specific system category and purpose.

## Template Selection Guide

Choose the appropriate template based on your system category:

| System Category | Template | Purpose |
|-----------------|----------|---------|
| Top-level Core Systems | [Main Index Template](mdc:dst-api-webdocs/dst-api-webdocs/#main-index-template) | Overall core-systems overview |
| Character Systems | [Character Systems Template](mdc:dst-api-webdocs/dst-api-webdocs/#character-systems-template) | Player/character related functionality |
| Game Mechanics | [Game Mechanics Template](mdc:dst-api-webdocs/dst-api-webdocs/#game-mechanics-template) | Gameplay systems and mechanics |
| Development Tools | [Development Tools Template](mdc:dst-api-webdocs/dst-api-webdocs/#development-tools-template) | Developer utilities and debugging |
| Technical Infrastructure | [Infrastructure Template](mdc:dst-api-webdocs/dst-api-webdocs/#infrastructure-template) | Low-level system components |

## Main Index Template

Use this template for the main core-systems/index.md file:

```markdown
---
id: core-systems-overview
title: Core Systems Overview
description: Overview of all core systems in the DST API
sidebar_position: 0

last_updated: 2024-XX-XX
build_version: 676042
change_status: stable
category_type: main-index
system_scope: all core systems
---

# Core Systems Overview

## Build Information
Current documentation based on build version: **676042**
Last updated: **2024-XX-XX**

## Core Systems Architecture

The DST API is organized into several interconnected core systems that provide the foundation for all game functionality. These systems work together to create the complete Don't Starve Together experience.

### System Categories

The core systems are organized into these major categories:

#### Game Foundation
Systems that provide the basic building blocks for all game functionality.

#### Player Experience  
Systems that directly impact how players interact with the game world.

#### Content Management
Systems that handle game assets, data, and content organization.

#### Development Support
Systems that assist with development, debugging, and maintenance.

## System Categories

### [Character Systems](mdc:dst-api-webdocs/dst-api-webdocs/character-systems/index.md)
Manages all aspects of character behavior, customization, and progression.

| System | Purpose | Key Components |
|--------|---------|----------------|
| [Core Character Systems](mdc:dst-api-webdocs/dst-api-webdocs/character-systems/core/index.md) | Base character functionality | Character utilities, player deaths |
| [Customization](mdc:dst-api-webdocs/dst-api-webdocs/character-systems/customization/index.md) | Character appearance and clothing | Beefalo clothing, player clothing |
| [Emotes](mdc:dst-api-webdocs/dst-api-webdocs/character-systems/emotes/index.md) | Character expressions | Emote items, emoji items |
| [Progression](mdc:dst-api-webdocs/dst-api-webdocs/character-systems/progression/index.md) | Character advancement | Skill trees, progression constants |
| [Speech](mdc:dst-api-webdocs/dst-api-webdocs/character-systems/speech/index.md) | Character dialogue | Character-specific speech patterns |

### [Data Management](mdc:dst-api-webdocs/dst-api-webdocs/data-management/index.md)
Handles all data persistence, assets, and file operations.

| System | Purpose | Key Components |
|--------|---------|----------------|
| [Assets](mdc:dst-api-webdocs/dst-api-webdocs/data-management/assets/index.md) | Asset loading and management | JSON handling, KLUMP files |
| [Saves](mdc:dst-api-webdocs/dst-api-webdocs/data-management/saves/index.md) | Save file operations | Save file upgrades, save indexing |
| [Utilities](mdc:dst-api-webdocs/dst-api-webdocs/data-management/utilities/index.md) | Data processing utilities | Platform post-load, scheduler |

### [Development Tools](mdc:dst-api-webdocs/dst-api-webdocs/development-tools/index.md)
Provides debugging, profiling, and development utilities.

| System | Purpose | Key Components |
|--------|---------|----------------|
| [Console](mdc:dst-api-webdocs/dst-api-webdocs/development-tools/console/index.md) | Console commands and reload | Console commands, hot reload |
| [Debugging](mdc:dst-api-webdocs/dst-api-webdocs/development-tools/debugging/index.md) | Debug utilities and helpers | Debug commands, debug keys |
| [Profiling](mdc:dst-api-webdocs/dst-api-webdocs/development-tools/profiling/index.md) | Performance analysis | Profiler, memory analysis |
| [Utilities](mdc:dst-api-webdocs/dst-api-webdocs/development-tools/utilities/index.md) | Development utilities | Dumper, string fixes |

### [Fundamentals](mdc:dst-api-webdocs/dst-api-webdocs/fundamentals/index.md)
Core building blocks that other systems depend on.

| System | Purpose | Key Components |
|--------|---------|----------------|
| [Actions](mdc:dst-api-webdocs/dst-api-webdocs/fundamentals/actions/index.md) | Player and entity actions | Action system, buffered actions |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vietnd69/dst-api-webdocs](https://github.com/vietnd69/dst-api-webdocs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
