---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a monorepo containing high-quality Slidev presentation templates. The primary template is **neko-style**, extracted from neko-talks and KubeCon HK 2025 presentations.

## Development Commands

### Running the Starter Demo
```bash
# From repository root
npm install
npm run dev:neko      # Start neko-style starter at http://localhost:3030
npm run build:neko    # Build for production
npm run export:neko   # Export to PDF/images
```

### Working with Individual Templates
```bash
# Navigate to template starter
cd neko-style/starter

# Development
npm run dev           # Start dev server with hot reload
npm run build         # Build static site
npm run export        # Export presentation
```

## Architecture

### Monorepo Structure
- **Root `package.json`**: Defines workspaces and top-level scripts
- **`neko-style/theme/`**: NPM theme package with components, layouts, and styles
- **`neko-style/starter/`**: Starter template that uses the theme
- **`neko-style/docs/`**: Shared documentation
- **`raw-template/neko-style/talks/packages/`**: Real-world presentation source packages

### neko-style Template Architecture

The template is organized as a **hybrid system** supporting two usage modes:

1. **Theme Package** (`theme/`): Installable via npm, provides components and layouts
2. **Starter Template** (`starter/`): Ready-to-use project template that references the theme

### neko-style Theme Architecture

**Core Component**: `components/GlowBackground.vue`
- Implements dynamic polygon-based gradient backgrounds
- Uses `seedrandom` for stable, reproducible random distributions
- Supports 3 color presets: `blue` (default), `rust`, `cyan`
- Reads frontmatter properties from each slide for per-page configuration

**Frontmatter Configuration System**:
Each slide in `slides.md` can configure its appearance via frontmatter:
```yaml
---
glowSeed: 42              # Seed for stable random background (required for variety)
glowPreset: blue          # Color preset: blue | rust | cyan
theme: dark               # Theme: dark | light
glow: center              # Distribution: full | top | bottom | left | right | center
glowOpacity: 0.4          # Background opacity (0-1)
glowHue: 0                # Hue shift (0-360)
---
```

**Component System**:
The template uses semantic, card-based components built with UnoCSS utility classes:
- **Color semantics**: red-800 (problems), green-800 (solutions), blue-800 (info), purple-800 (advanced), yellow-800 (performance)
- **Card structure pattern**: Border + background with header and content sections
- **Animation system**: Unified 500ms transitions with `v-click` for progressive disclosure

**Styling Architecture**:
- `theme/styles/index.css`: Base styles and transition conventions
- `starter/uno.config.ts`: UnoCSS configuration used by the starter template
- Uses UnoCSS with presets: Uno, Attributify, Icons, WebFonts
- Icon collections: `@iconify-json/carbon`, `@iconify-json/logos`

## Key Files and Their Purposes

- **`neko-style/docs/FOR-AI-ASSISTANTS.md`**: Comprehensive guide for AI assistants creating presentations with this template
- **`neko-style/docs/components-guide.md`**: Complete component library with copy-paste code examples
- **`neko-style/docs/QUICK-START.md`**: 5-minute quick start guide
- **`neko-style/theme/components/GlowBackground.vue`**: Core glow background implementation
- **`neko-style/theme/global-bottom.vue`**: Theme-level global background entry for Slidev
- **`neko-style/theme/styles/index.css`**: Theme base styles
- **`neko-style/starter/uno.config.ts`**: Starter-side UnoCSS configuration

## Creating New Presentations

When helping users create presentations with neko-style:

1. **Read the AI guide first**: Always reference `neko-style/docs/FOR-AI-ASSISTANTS.md` for detailed instructions
2. **Copy components from guide**: Use `neko-style/docs/components-guide.md` as the source of truth for component code
3. **Follow semantic colors**: Red for problems, green for solutions, blue for information
4. **Vary glowSeed per page**: Use different seed values (100, 150, 200, etc.) for visual variety
5. **Use v-click for animations**: Wrap content in `<v-click>` for progressive disclosure

## Template Setup for New Projects

### Option 1: Use Starter Template (Recommended for new users)

```bash
npx degit user/repo/neko-style/starter my-presentation
cd my-presentation
npm install
npm run dev
```

### Option 2: Install Theme Package from Local Path (Recommended for existing projects)

> `slidev-theme-neko-style` is currently not published to npm registry.

```bash
# clone template repo first
git clone https://github.com/iridite/slidev-templates.git

# install local theme package in your Slidev project
npm install /absolute/path/to/slidev-templates/neko-style/theme
```

Then add to `slides.md`:
```yaml
---
theme: neko-style
---
```

## Theme Development

When modifying the theme itself:

- **Theme source**: All components, layouts, and styles are in `neko-style/theme/`
- **Test with starter**: Use `neko-style/starter/` to test theme changes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iridite/slidev-templates](https://github.com/iridite/slidev-templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
