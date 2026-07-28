---
trigger: always_on
description: OctoCanvas is a GitHub-themed application that creates two types of collectibles based on GitHub user profiles:
---

# OctoCanvas - Custom Instructions

## Project Overview
OctoCanvas is a GitHub-themed application that creates two types of collectibles based on GitHub user profiles:
1. **Wallpaper Generator**: Personalized wallpapers with contribution graphs
2. **Devémon Card**: Trading card-style collectibles with GitHub stats and rarity system

Built with Astro, Preact, TypeScript, and Tailwind CSS.

## Design Systems

### GitHub Universe Theme (Wallpaper Generator)
- **Primary Color**: `#5fed83` (Universe Green)
- **Secondary Colors**: 
  - `#08872B` (Dark Green)
  - `#bfffd1` (Light Green)
  - `#dcff96` (Pale Green)
  - `#1a7f37` (Medium Green)
- **Typography**: 
  - Font: Monaspace Neon (monospace)
  - Letter spacing: 2px
  - All uppercase for headers and labels
- **Effects**: 
  - Box shadows with 0.5px/-0.5px offsets
  - Gradient borders
  - Glow effects on hover

### GitHub Primer Colors (Devémon Card)
- **Green**: `#7ee787` (success, grass type)
- **Blue**: `#58a6ff` (water type)
- **Purple**: `#bc8cff` (psychic type)
- **Orange**: `#ffa657` (fire type)
- **Red**: `#ff7b72` (danger, dragon type)
- **Gray**: `#7d8590` (neutral, muted text)

### Color Palette (Tailwind Config)
```
universe-green: #5fed83
universe-green-light: #bfffd1
universe-green-hover: #4dd46e
universe-green-dark: #08872B
universe-green-border: #a8e6c5
universe-black: #0a0e0d
universe-dark-bg: #101411
universe-dark-surface: #1c201e
universe-dark-border: #2d3330
universe-gray-muted: #8b918f
```

## Core Features

### Feature 1: Wallpaper Generator

#### GitHub Profile Integration
- Fetches user data via GitHub API
- Displays:
  - Avatar (with CORS-safe base64 conversion)
  - Username and display name
  - Follower count
  - Repository count
  - **Total contributions** (accurate count from last 12 months)
  - Bio (if available)

#### Contribution Graph
- **Data Source**: Fetches contribution data via API (`https://contributions-api.me-5bd.workers.dev/?username=${encodeURIComponent(username)}`)
- **Visualization**: 
  - 53-week calendar (last year)
  - Color intensity based on contribution count:
    - 0 contributions: `#101411` (dark, 0.3 opacity)
    - 1-2 contributions: `#dcff96` (pale green, 0.35 opacity)
    - 2-5 contributions: `#bfffd1` (light green, 0.5 opacity)
    - 5-10 contributions: `#8cf2a6` (medium green, 0.6 opacity)
    - 10-15 contributions: `#5fed83` (bright green, 0.8 opacity)
    - 15+ contributions: `#5fed83` (bright green, 1.0 opacity)
  - Cell size: 12px (scaled), gap: 3px (scaled)
  - Properly centered on wallpaper

#### Wallpaper Sizes & Export
- **SVG-based**: Vector graphics with responsive scaling
- **Three Sizes**:
  - Desktop: 2560×1440 (16:9)
  - Mobile: 1170×2532 (portrait)
  - Badge: 320×240 (landscape)
- **Export**: PNG format via Canvas API
- **Content**:
  - Centered avatar with green border
  - User stats (followers, repos, contributions)
  - Contribution graph at bottom
  - Decorative floating circles
  - GitHub Universe branding

#### Animated Preview
- **AnimatedBackground Component**: 
  - 50 particles with random positions and velocities
  - Radial gradient circles (Universe Green)
  - Gentle floating animation
  - 0.4 opacity for subtle effect
  - Visible in black areas around wallpaper preview
- **SVG Animations** (preview only, not in downloads):
  - Pulsing glow on avatar border (3s cycle)
  - Floating decorative circles (6s cycle, staggered delays)
  - Fade-in animations on text and contribution squares
  - Smooth entrance effects
- **Responsive Preview**:
  - Mobile (<768px): Shows portrait wallpaper (9:19.5 aspect ratio)
  - Desktop (≥768px): Shows landscape wallpaper (16:9 aspect ratio)
  - Auto-adjusts on window resize

### Feature 2: Devémon Card (Trading Card System)

#### Card Formats
- **Card Format**: 350×550px (portrait, 7:11 aspect ratio)
  - Holographic border effect
  - Avatar at top center (96×96px)
  - Stats displayed below avatar
  - Power calculation shown
  - Rarity badge
  - Type indicator with icon
  - "Available for Hire" badge (optional)
  
- **Badge Format**: 320×240px (landscape, 4:3 aspect ratio)
  - Compact horizontal layout
  - Avatar left side (80×80px)
  - Stats in center column
  - Contribution graph visualization
  - Rarity indicator
  - "Available for Hire" badge (optional, bottom placement)

#### Rarity System
Power is calculated based on GitHub stats:
```
power = (followers × 10) + (repos × 5) + (totalContributions × 0.1)
```

Rarity levels based on power thresholds:
1. **Common** (0-99): Gray (`#7d8590`) - "Normal" type, ⭐
2. **Uncommon** (100-499): Green (`#7ee787`) - "Grass" type, ⭐⭐
3. **Rare** (500-999): Blue (`#58a6ff`) - "Water" type, ⭐⭐⭐
4. **Epic** (1000-4999): Purple (`#bc8cff`) - "Psychic" type, 🌟
5. **Legendary** (5000-9999): Orange (`#ffa657`) - "Fire" type, 🌟🌟
6. **Mythical** (10000+): Red (`#ff7b72`) - "Dragon" type, 🌟🌟🌟

#### Available for Hire Badge
- **Toggle**: Checkbox control to enable/disable badge
- **Display**: 
  - Card format: "💼 OPEN TO WORK" badge below avatar
  - Badge format: "💼 OPEN TO WORK" badge at bottom of layout
- **Styling**: 
  - Green background (`#7ee787`)
  - Dark text for contrast
  - Small font size (9px for badge format)
  - Rounded corners with padding

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [github/octocanvas](https://github.com/github/octocanvas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
