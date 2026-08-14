---
trigger: always_on
description: This document provides comprehensive guidance for AI assistants working on the duv0-x.github.io repository.
---

# CLAUDE.md - AI Assistant Guide

This document provides comprehensive guidance for AI assistants working on the duv0-x.github.io repository.

## Repository Overview

**Type**: Personal Portfolio Website (GitHub Pages)
**Owner**: Duván Ballén (duv0_x)
**Tech Stack**: Static HTML/CSS
**Purpose**: Professional profile and activity log showcasing real daily work across SRE/DevOps, cloud engineering, personal projects, and AI tooling.
**Theme**: Retro pixel-art / terminal aesthetic with dark color scheme; log entries rendered as terminal lines with `>>` prompt

## Codebase Structure

```
duv0-x.github.io/
├── index.html          # Main landing page (single-page site)
├── styles.css          # All styling and theme definitions
├── README.md           # Repository description
├── .gitignore          # Git ignore patterns
├── files/              # CV/Resume PDFs in multiple versions
│   ├── CV_Duvan_Ballen_SRE_DevOps.pdf
│   ├── CV_Duvan_Ballen_SRE_DevOps_EN.pdf
│   ├── CV_Duvan_Ballen_SRE_DevOps_EN_v2.pdf
│   └── CV_Duvan_Ballen_SRE_DevOps_v2.pdf
└── images/             # Profile pictures and visual assets
    ├── duv0-x-px.png
    ├── duv0-x-small.jpg
    ├── duv0-x.jpg
    └── x-0vub-small.jpg
```

## Design & Styling Conventions

### Color Palette
The site uses a **consistent dark theme** with specific brand colors:

- **Background**: `#000` (black body) and `#0c1015` (container backgrounds)
- **Primary Accent**: `#27a888` (teal/green - used for headers)
- **Secondary Accent**: `#98d1cc` (light teal - used for text and borders)
- **Link Colors**:
  - `#e06c76` (coral/pink - LinkedIn)
  - `#98d1cc` (light teal - GitHub, Mastodon)

**CRITICAL**: When making any color changes:
1. Maintain visual consistency across the site
2. Ensure sufficient contrast for readability
3. Test on both light and dark backgrounds
4. Update all instances of a color if changing brand colors

### Typography
- **Primary Font**: "Press Start 2P" (retro pixel font from Google Fonts)
- **Base Font Size**: 10px (very small, pixelated aesthetic)
- **Footer Font Size**: 8px
- **Line Height**: 1.5

### Visual Style
- **Pixel-art aesthetic**: Uses `image-rendering: pixelated` for images
- **Border style**: 4px solid borders with background colors
- **Spacing**: Minimal margins (5px) for compact layout
- **Box shadows**: Subtle `0px 0px 10px rgba(0, 0, 0, 0.1)`
- **Border radius**: 10px for rounded corners on images

## HTML Structure Conventions

### Page Sections (index.html:13-47)
The page is divided into semantic containers:

1. **Header** (`<header>`) - Site title/branding
2. **About Container** (`.about-container`) - Profile image and personal info
3. **Tech Stack** (`.section` with `.section-title`) - Pixelated pills derived from post hashtags, styled as a terminal package list
4. **Work Log** (`.section` with `#work-log`) - Terminal-style log entries with pagination (5 per page)
5. **Links** (`.section` with `.section-title`) - Professional and social links
6. **Footer** (`<footer>`) - Build credits

### Log Entries

Posts use `.log-entry` containers with:
- `.log-line` - One terminal line containing:
  - `.log-prompt` - `>>` prefix in coral accent color
  - `.log-timestamp` - Date stamp in primary accent color
  - `.log-message` - Activity description in secondary accent color
- `.log-tags` - Hashtags in primary accent color

The Work Log uses vanilla JavaScript for pagination: 5 entries per page, with `prev`/`next` buttons.

New posts are added by duplicating an existing `<article class="log-entry">` block inside `#log-entries`. When a new technology hashtag is introduced, add the corresponding `.stack-pill` to the Tech Stack cloud. Remember to keep the total entry count aligned with the pagination logic (`pageSize = 5` in the inline script).

### Layout Patterns
- **Flexbox for about section**: `display: flex` with `align-items: center` and `gap: 10px`
- **Max-width containers**: 500px for readability
- **Responsive images**: `max-width: 80%` on images
- **Text alignment**: Left-aligned in about section, center-aligned elsewhere

## Development Workflows

### Git Workflow

**Branch Naming Convention**:
- Feature branches: `claude/descriptive-name-xxxxx`
- Branch names must start with `claude/` for AI assistant work
- Must include session ID suffix for push authentication

**Commit Message Style** (based on recent commits):
- Use descriptive, imperative mood messages
- Examples:
  - "Color refactor"
  - "Some links deletion"
  - "Change theme colors"
  - "Update index.html" (for minor changes)

**Pull Request Pattern**:
- Create feature branches for all changes
- Merge via pull requests (even for small changes)
- Clean commit history with merge commits
- Recent PRs: #8 (links-deletion), #7 (color-refactor), #6 (change-colors)

### Testing Checklist

Before committing changes to this **static site**, verify:

1. **HTML Validation**: Ensure valid HTML5 syntax
2. **CSS Validation**: Check for syntax errors in styles.css
3. **Visual Testing**:
   - Check layout at different viewport sizes
   - Verify color consistency across all sections
   - Test all links (external APIs, social links)
4. **Cross-browser Testing**: Test on major browsers
5. **External Resources**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duv0-x/duv0-x.github.io](https://github.com/duv0-x/duv0-x.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
