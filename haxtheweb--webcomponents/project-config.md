---
trigger: always_on
description: This file provides comprehensive instructions for AI coding agents working within the HAX ecosystem. HAX (Headless Authoring eXperience) is a comprehensive web development ecosystem that enables rapid creation of accessible, performant web components and static sites. Follow these guidelines to set up, develop, test, and contribute effectively to HAX projects.
---

# AGENTS.md

This file provides comprehensive instructions for AI coding agents working within the HAX ecosystem. HAX (Headless Authoring eXperience) is a comprehensive web development ecosystem that enables rapid creation of accessible, performant web components and static sites. Follow these guidelines to set up, develop, test, and contribute effectively to HAX projects.

## HAX Ecosystem Overview

The HAX ecosystem consists of multiple interconnected repositories, each serving specific purposes:

### Core Repositories

- **`webcomponents`** - The heart of HAX: a monorepo containing 250+ LitElement-based web components, themes, and the DDD design system. All components are built with accessibility, performance, and HAX compatibility in mind.

- **`create` (@haxtheweb/create)** - The HAX CLI tool for scaffolding new web components, HAXsites, and managing the development workflow. This is your primary interface for creating new HAX projects.

- **`haxcms-php`** - PHP backend implementation of HAXcms, providing content management capabilities, API endpoints, and server-side rendering for HAXsites.

- **`haxcms-nodejs`** - Node.js backend implementation of HAXcms, offering the same capabilities as the PHP version but in a JavaScript environment.

- **`desktop`** - Electron-based desktop application that provides a local development environment for HAX, combining the power of HAXcms with desktop convenience.

### Supporting Repositories

- **`hax11ty`** - Integration layer that bridges HAX components with Eleventy (11ty) static site generator, enabling HAX components in traditional static sites.

- **`json-outline-schema`** - Defines the JSON schema used by HAXcms for content structure, site navigation, and metadata management.

- **`hax-schema`** - Contains HAX property schemas that define how web components integrate with the HAX authoring interface.

- **`open-apis`** - Microservice APIs and shared infrastructure for HAXTheWeb ecosystem. Contains Express-based Vercel endpoints deployed at https://open-apis.hax.cloud/ providing advanced capabilities like content importing, parsing, analysis, site conversion, and migration services. Also hosts the pre-built Storybook documentation for HAX components.

- **`docs`** - Official HAX documentation site built as a HAXcms site. Contains comprehensive documentation about HAX philosophy, pillars, community guidelines, tutorials, and developer resources. Structure follows HAXcms conventions with site.json (JSON Outline Schema), pages/ directory for content, and files/ for assets.

- **`issues`** - Unified issue tracking repository for the entire HAX ecosystem, where all bugs, features, and discussions are centralized.

### Development Philosophy & Community Pillars

HAX is built on a foundation of community pillars that guide all development decisions and community interactions. These are not just technical specifications but ethical commitments:

#### Core Pillars
- **Accessible**: HAX maximizes accessibility while removing the knowledge required to maintain accessibility standards. Components adhere to WCAG 2.0 AA standards, with automated color contrast ratios and expert-audited implementations.
- **Extensible**: Built for sustainable extension through web standards, microservices, and modular architecture that grows with user needs.
- **Free and Open**: Open community welcoming all (Penn State and beyond), embracing the 5Rs of OER (Retain, Reuse, Revise, Remix, Redistribute) for content and code.
- **Efficient**: Optimized for performance through web standards over heavy libraries, lazy loading, offline capability, and minimal resource usage.
- **Platform Agnostic**: Works anywhere - standalone HAXsites, integrated HAXcms, static web pages, or existing CMS platforms.
- **Remixable**: Maximizes remix-ability through modular design, open licensing, and semantic content structures.
- **Sustainable**: Environmental (less data, lower battery usage), technological (web standards-based longevity), and community (inclusive, collaborative governance) sustainability.

#### Technical Emphasis
- **Rapid Development**: Scaffolding tools and design systems accelerate creation
- **Unbundled Delivery**: Pure JavaScript, HTML, CSS approach without compilation steps
- **Modularity**: Components work independently and compose together seamlessly
- **Ubiquitous Web**: The web needs a file format - content should "just work" regardless of how it was built

## Development Environment Setup

### Issue Management
Before starting any work, check the unified issue queue:
- All HAX ecosystem issues are tracked at `haxtheweb/issues`
- Use GitHub CLI to check issues: `gh issue list`
- Reference existing issues when making contributions

## Setup Commands

- **Install HAX CLI globally**:
  ```bash
  npm install @haxtheweb/create --global
  ```
- **Install project dependencies** (run from the project root):
  ```bash
  npm install
  ```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haxtheweb/webcomponents](https://github.com/haxtheweb/webcomponents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
