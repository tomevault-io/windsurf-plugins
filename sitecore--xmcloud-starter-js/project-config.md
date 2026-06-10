---
trigger: always_on
description: XM Cloud Starter repository context and constraints
---


# Project Context

## Repository Overview

This is the **XM Cloud Front End Application Starter Kits** repository containing multiple Next.js starter applications and SPA examples for Sitecore XM Cloud development.

Repository Structure:
- `/examples/` - Contains starter front-end applications (Next.js and SPA)
- `/authoring/` - Sitecore content items, templates, and deployment configurations  
- `/local-containers/` - Docker setup for local development environments
- `xmcloud.build.json` - Primary configuration for XM Cloud deployment

## Starter Applications

Available Examples:
- `basic-nextjs` - Simple Next.js starter with basic XM Cloud integration
- `kit-nextjs-article-starter` - **Solterra & Co.** - Editorial-style template for lifestyle brands
- `kit-nextjs-location-finder` - **Alaris** - Car brand template with location finder functionality
- `kit-nextjs-product-listing` - **SYNC** - Product-focused template for audio gear companies
- `kit-nextjs-skate-park` - Simple demo site showcasing component examples
- `basic-spa` - SPA starter kit with Angular and Node proxy

Each starter demonstrates:
- Tailwind-based styling with Shadcn/ui components
- Personalized homepage via URL parameters
- Modular component architecture with variants
- Localization support for English (en) and Canadian English (en-CA)

## Technology Stack

Core Technologies:
- **Next.js 14+** - React framework with App Router and Pages Router support
- **TypeScript** - Strict type safety throughout all components
- **Sitecore XM Cloud** - Headless content management and delivery
- **Sitecore Content SDK** - Modern SDK for XM Cloud integration
- **Tailwind CSS** - Utility-first CSS with container queries (@container)
- **Shadcn/ui** - Modern component library with accessibility features

Additional Libraries:
- **Framer Motion** - Animation library for interactive components
- **Lucide React** - Icon library for consistent iconography  
- **next-localization** - Internationalization with dictionary support
- **change-case** - String case transformation utilities

Development Tools:
- **Docker** - Containerized local development with Sitecore CM
- **Node.js LTS** - JavaScript runtime environment
- **npm** - Package management across all starter applications

## Development Principles

Multi-Starter Architecture:
- Each example is a standalone application
- Shared patterns and conventions across all starters
- Independent deployment and development workflows
- Common XM Cloud integration patterns

Content-First Development:
- Components are designed around Sitecore data structures
- Field-driven rendering with proper fallbacks
- Support for both connected and disconnected development modes
- Proper handling of content authoring scenarios

## Upstream repository, forks, and pull request scope

Before you plan work that is intended as a **pull request**, determine **which repository is the target**: the **official upstream** (e.g. the public Sitecore `xmcloud-starter-js` repo) or a **user fork / private copy** the user uses as their own app.

**Why this matters:** upstream maintains a **small, fixed set of starters** under `examples/` as **reference examples** that follow shared best practices—not an open collection of every vertical or use case.

**Contributions to upstream (acceptable):**
- **Improvements, bug fixes, and broadly useful features** in **existing** starters
- **Documentation** and **shared tooling** that help those starters, consistent with the repo’s contribution rules

**Do not propose or prepare pull requests to upstream** for:
- **New example sites** or **additional** starter projects under `examples/`
- **Bespoke extensions** of a starter for **one org’s** specific product or business requirements (those belong in a fork)
- Changes that exist mainly to support a **single user’s** long-term product rather than a **reusable, general** improvement to the public examples

**Encouraged on the user’s own fork (not as upstream PRs):**
- **Adding more starters** or new example applications
- **Heavily customizing** a starter for a **specific** deployment or product

**Agent workflow**
1. **Clarify the target** (ask if needed, or use remotes, repo URL, workspace context): **upstream** vs **fork/standalone** repository.
2. If the target is **upstream**: do **not** create PRs that add **new example sites**, new `examples/*` applications, or fork-style bespoke scope; keep proposed PRs to **improvements / fixes / generally useful features** in **existing** starters.
3. If the user works in **their own fork or template copy**: they may add starters and extend freely; align the work with **that** repository and avoid framing fork-only changes as an upstream PR.

For human-readable policy, see @CONTRIBUTING.md and the [What we do not accept](CONTRIBUTING.md#what-we-do-not-accept) section.

## Constraints and Guidelines

Starter Independence:
- Each starter is a standalone application. Copy shared utilities and components into the starter; do not create shared packages or symlinks across starters. Each starter can be run and maintained independently.

File Organization:
- Each starter maintains its own `src/` directory structure
- Shared utilities should be copied, not shared (no monorepo linking)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sitecore/xmcloud-starter-js](https://github.com/Sitecore/xmcloud-starter-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
