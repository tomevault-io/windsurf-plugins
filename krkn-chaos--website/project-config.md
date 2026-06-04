---
trigger: always_on
description: <img src="./static/img/logo.png" alt="Krkn - Chaos Engineering for Kubernetes" width="100%"/>
---

<p align="center">
  <img src="./static/img/logo.png" alt="Krkn - Chaos Engineering for Kubernetes" width="100%"/>
</p>

<p align="center">
  <a href="https://github.com/krkn-chaos/website/releases"><img src="https://img.shields.io/github/v/release/krkn-chaos/website?style=flat-square&color=0E58A0&label=release" alt="Release"></a>
  <a href="https://github.com/krkn-chaos/website/stargazers"><img src="https://img.shields.io/github/stars/krkn-chaos/website?style=flat-square&color=EC1C24" alt="Stars"></a>
  <a href="https://github.com/krkn-chaos/website/network/members"><img src="https://img.shields.io/github/forks/krkn-chaos/website?style=flat-square&color=334155" alt="Forks"></a>
  <a href="https://krkn-chaos.dev"><img src="https://img.shields.io/badge/docs-krkn--chaos.dev-3B82F6?style=flat-square" alt="Documentation"></a>
</p>

---

# CLAUDE.md - Krkn Website

## Project Overview

The [krkn-chaos.dev](https://krkn-chaos.dev) website is the documentation and marketing site for Krkn, a chaos engineering tool for Kubernetes/OpenShift. Built with Hugo + Docsy theme, deployed on Netlify. Apple-inspired dark/light theme with custom SCSS architecture.

## Repository Structure

```
website/
├── content/en/                  # All English content (Markdown + HTML)
│   ├── docs/                    # Documentation section
│   │   ├── scenarios/           # 20+ chaos scenario pages (each with tab content)
│   │   ├── krkn/                # Core Krkn docs
│   │   ├── krknctl/             # CLI tool docs
│   │   ├── krkn_ai/             # Krkn AI docs (config/, getting-started, etc.)
│   │   ├── installation/        # Installation guides
│   │   ├── getting-started/     # Quick start
│   │   ├── cerberus/            # Health monitoring
│   │   ├── developers-guide/    # Developer documentation
│   │   ├── contribution-guidelines/ # How to contribute
│   │   └── chaos-testing-guide/ # Best practices
│   ├── blog/                    # Blog section (curated external links)
│   └── community/               # Community page
├── layouts/                     # Hugo template overrides
│   ├── index.html               # Custom homepage (hero, features, scenarios, CTA)
│   ├── _partials/               # Docsy partial overrides (MUST use underscore prefix)
│   │   ├── navbar.html          # Custom glass-morphism navbar
│   │   ├── footer.html          # Custom 4-column footer
│   │   ├── chatbot.html         # AI chatbot widget
│   │   └── hooks/
│   │       ├── head-end.html    # Theme init script (prevents FOUC)
│   │       └── body-end.html    # Scroll animations, chatbot inject, tab hash sync
│   ├── blog/                    # Custom blog layout (replaces Docsy default)
│   │   ├── baseof.html
│   │   └── list.html
│   ├── community/               # Custom community layout (replaces Docsy default)
│   │   ├── baseof.html
│   │   └── list.html
│   ├── shortcodes/              # Custom shortcodes (notice, include)
│   └── _default/_markup/        # Heading anchor renderer
├── assets/scss/                 # SCSS source files
│   ├── _styles_project.scss     # Main entry — CSS custom properties + imports
│   ├── _variables_project.scss  # Bootstrap/Docsy variable overrides
│   ├── _custom_animations.scss  # Scroll-triggered reveal animations
│   ├── _custom_navbar.scss      # Navbar: glass-morphism, responsive, theme toggle
│   ├── _custom_hero.scss        # Hero section: gradient, cluster animation, CTAs
│   ├── _custom_sections.scss    # Feature grid, scenarios, timeline, CTA, footer
│   ├── _custom_docs.scss        # Documentation: sidebar, content, TOC, code blocks
│   └── _custom_pages.scss       # Blog & community page styles
├── static/                      # Static assets (CSS, JS, images, favicons)
│   ├── css/                     # Chatbot CSS, tab scroll CSS
│   ├── js/                      # Scroll animations JS, chatbot JS
│   ├── img/                     # Logos (Krkn SVG, CNCF white/color)
│   └── search-index.json        # Pre-built Lunr search index
├── netlify/functions/           # Netlify serverless functions (chat, health, etc.)
├── hugo.yaml                    # Main Hugo configuration
├── package.json                 # npm dependencies and build scripts
├── go.mod                       # Go module (Docsy v0.12.0)
├── netlify.toml                 # Netlify deployment configuration
└── .github/workflows/           # CI (rebuild-docs-index on content changes)
```

## Quick Start

```bash
# Install dependencies
npm install

# Development server (with drafts and future posts)
npm run dev
# or
hugo server --disableFastRender

# Production build
npm run build:production

# Clean build artifacts
npm run clean
```

## Stack & Versions

- **Hugo:** v0.146.0+ (extended edition, required for SCSS)
- **Docsy theme:** v0.12.0 (imported as Hugo module via `go.mod`)
- **Go:** 1.21+ (for Hugo modules)
- **Node:** 20 (for build scripts and Netlify functions)
- **Deployment:** Netlify (auto-deploys from `main` branch)
- **Search:** Lunr.js offline search (`static/search-index.json` built at build time)
- **Fonts:** Satoshi (headings) + General Sans (body) via Fontshare CDN

## Hugo Configuration

- **Main config:** `hugo.yaml`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [krkn-chaos/website](https://github.com/krkn-chaos/website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
