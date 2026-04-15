---
trigger: always_on
description: A personal portfolio website for **Matthew McWilliams** built with React and Vite. The site is fully static (no backend) for GitHub Pages hosting. Features a dark theme with blue-cyan gradient aesthetic, interactive aurora background with particles, and a polished 3D carousel showcase for projects.
---

# Portfolio Website - Project Instructions

## Project Overview

A personal portfolio website for **Matthew McWilliams** built with React and Vite. The site is fully static (no backend) for GitHub Pages hosting. Features a dark theme with blue-cyan gradient aesthetic, interactive aurora background with particles, and a polished 3D carousel showcase for projects.

**Project Size**: Large - requires detailed planning, version control, and comprehensive documentation.

---

## Tech Stack

- **Framework**: React 18+ with Vite
- **Styling**: Tailwind CSS with centralized CSS custom properties
- **Animation**: Motion (Framer Motion) for advanced animations
- **Build Output**: Static files for GitHub Pages deployment
- **No external backend** - all data is local/static

---

## Design System

### Color Palette
- **Primary Blue**: `#0061FF`
- **Primary Cyan**: `#60EFFF`
- **Gradient**: Linear gradient from blue to cyan
- **Background**: Pure black (`#000000`) with aurora overlay
- **Glass Effects**: `rgba(0, 0, 0, 0.7)` with blur for cards/navbar
- **Text**: White with various opacity levels for hierarchy

### Typography
- **Primary Font**: Orbitron (futuristic, techy aesthetic)
- **Fallback**: Inter, system-ui, sans-serif
- **Letter Spacing**: 0.06em globally for readability
- **Hero Text**: Orbitron bold for main headings

### CSS Architecture
All design tokens are centralized in `src/styles/index.css` using CSS custom properties:
- `--color-primary-blue`, `--color-primary-cyan`
- `--gradient-primary`, `--gradient-primary-subtle`
- `--text-primary`, `--text-secondary`, `--text-muted`
- `--transition-fast`, `--transition-base`, `--transition-slow`

---

## Interactive Components

### MagneticButton
Located in `src/components/ui/MagneticButton.jsx`

Features:
- **Magnetism**: Button subtly follows cursor (0.15 strength)
- **Animated Border**: Conic gradient draws around button on hover (starts top-left at 287.5deg)
- **Shine Effect**: Radial gradient follows cursor position
- **Scale**: Slight size increase on hover (1.05x)
- **Colors**: Blue-cyan gradient border, black background with gradient overlay

### NavLink
Located in `src/components/ui/NavLink.jsx`

Features:
- **Magnetism**: Same as MagneticButton
- **Shine Effect**: Radial gradient on hover
- **Active State**: Gradient background only on current page

### DecryptedText
Located in `src/components/ui/DecryptedText.jsx`

Features:
- **Scramble Animation**: Characters randomly scramble before revealing
- **Sequential Reveal**: Characters reveal one by one from start
- **useOriginalCharsOnly**: Only uses letters from the original text
- **animateOn="both"**: Animates on page load AND re-animates on hover
- **onComplete callback**: Triggers when animation finishes (deferred via `setTimeout` to avoid React render-cycle warnings)

### InterestTile
Located in `src/components/ui/InterestTile.jsx`

Features:
- **Magnetism**: Tile follows cursor on hover
- **3D Tilt**: Perspective tilt based on cursor position
- **Glow**: Border glow effect on hover
- No shine overlay (removed for cleaner visuals)

### ProjectCard
Located in `src/components/projects/ProjectCard.jsx`

Features:
- **Magnetism**: Card follows cursor on hover
- **3D Tilt**: Perspective tilt based on cursor position
- **Glow**: Border glow effect on hover
- **Click to Navigate**: Opens project URL
- No shine overlay (removed for cleaner visuals)

---

## Site Structure

### Header/Navbar
- **Position**: Absolute (not fixed), hides when scrolled past 50px
- **Style**: Floating card with rounded corners, glass effect
- **Logo**: "Matthew McWilliams" with gradient text and magnetism effect
- **Nav Links**: Interactive with magnetism and shine
- **Mobile**: Hamburger menu with same interactive effects

### Page 1: Home (`/`)
- **Hero Text**: "Hey, I'm Matthew." with DecryptedText animation
- **Buttons**: Fade in after text animation completes
- **Button Style**: MagneticButton with full effects
- **Text moves up** when buttons appear for visual flow

### Page 2: About (`/about`)
- **Header**: "About Me" with `page-title-short` gradient class
- **Bio Card**: Glass effect card with personal description
- **Interest Tiles**: 7 tiles (Programming, Gaming, Travel, 3D Printing, AI, Music, Rubik's Cubes)
- **GitHub Button**: MagneticButton linking to GitHub profile

### Page 3: Projects (`/projects`)
- **Header**: "My Stuff" with `page-title-short` gradient class
- **3D Carousel**: 5 visible cards with perspective transforms
- **Card Interactions**: Tilt on hover, click to navigate
- **Scroll/Swipe**: Rotates carousel with snap behavior

### Page 4: Contact (`/contact`)
- **Header**: "Get In Touch" with gradient
- **Contact Cards**: Email and GitHub with icons

---

## File Structure

```
Portfolio/
├── public/
│   ├── favicon.svg
│   └── images/projects/
├── src/
│   ├── components/
│   │   ├── layout/
│   │   │   ├── Navbar.jsx (with logo magnetism)
│   │   │   └── ParticleBackground.jsx (aurora + particles)
│   │   ├── ui/
│   │   │   ├── Button.jsx

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/matmcw) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
