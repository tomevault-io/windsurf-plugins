---
trigger: always_on
description: - [Repository Overview](#repository-overview)
---

# Project Context

## Table of Contents

- [Repository Overview](#repository-overview)
- [Technology Stack](#technology-stack)
- [Development Principles](#development-principles)
- [Upstream, forks, and pull request scope](#upstream-forks-and-pull-request-scope)
- [Constraints and Guidelines](#constraints-and-guidelines)
- [Code Style](#code-style)
- [General Coding Principles](#general-coding-principles)
- [JavaScript/TypeScript Rules](#javascripttypescript-rules)
- [Sitecore XM Cloud Rules](#sitecore-xm-cloud-rules)
- [Next.js Development Patterns](#nextjs-development-patterns)
- [Testing Patterns](#testing-patterns)
- [Safety Rules](#safety-rules)

## Repository Overview

This is the **XM Cloud Front End Application Starter Kits** repository containing multiple Next.js starter applications and SPA examples for Sitecore XM Cloud development.

**Repository Structure:**
- `/examples/` - Contains starter front-end applications (Next.js and SPA)
- `/authoring/` - Sitecore content items, templates, and deployment configurations  
- `/local-containers/` - Docker setup for local development environments
- `xmcloud.build.json` - Primary configuration for XM Cloud deployment

**Available Examples:**
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

**Core Technologies:**
- **Next.js 14+** - React framework with App Router and Pages Router support
- **TypeScript** - Strict type safety throughout all components
- **Sitecore XM Cloud** - Headless content management and delivery
- **Sitecore Content SDK** - Modern SDK for XM Cloud integration
- **Tailwind CSS** - Utility-first CSS with container queries (@container)
- **Shadcn/ui** - Modern component library with accessibility features

**Additional Libraries:**
- **Framer Motion** - Animation library for interactive components
- **Lucide React** - Icon library for consistent iconography  
- **next-localization** - Internationalization with dictionary support
- **change-case** - String case transformation utilities

**Development Tools:**
- **Docker** - Containerized local development with Sitecore CM
- **Node.js LTS** - JavaScript runtime environment
- **npm** - Package management across all starter applications

## Development Principles

**Multi-Starter Architecture:**
- Each example is a standalone application
- Shared patterns and conventions across all starters
- Independent deployment and development workflows
- Common XM Cloud integration patterns

**Content-First Development:**
- Components are designed around Sitecore data structures
- Field-driven rendering with proper fallbacks
- Support for both connected and disconnected development modes
- Proper handling of content authoring scenarios

## Upstream, forks, and pull request scope

The official **upstream** repository (this project’s public GitHub home) keeps a **small, fixed** set of starters in `examples/` as **reference examples**, not a catalog of every vertical. Before treating work as an **upstream pull request**, confirm whether the target is **upstream** or a **user fork** / **template** copy.

- **Fits upstream PRs:** **Improvements, bug fixes, and broadly useful features** in **existing** starters; **documentation** and **tooling** aligned with the repo’s contribution policy.
- **Not for upstream PRs:** **New example sites**, **additional** starters, or **product-specific** extensions for one org that should live in a **fork** or separate repo. Use **Use this template** or maintain a **fork** for that work.
- **Fork or standalone copy:** Add starters and customize freely; do not frame that work as an official upstream change unless maintainers have agreed otherwise.

Authoritative human policy: **[CONTRIBUTING.md](CONTRIBUTING.md)** — especially **[What we do not accept](CONTRIBUTING.md#what-we-do-not-accept)**. For Cursor, see **`.cursor/rules/project-context.mdc`**.

## Constraints and Guidelines

**File Organization:**
- Each starter maintains its own `src/` directory structure
- Shared utilities should be copied, not shared (no monorepo linking)
- Configuration files specific to each starter application
- Independent package.json for each example

**Development Workflow:**

DMZ git flow will be implemented in the future to support better development practices, scaling, efficiency and developer productivity.
Below is an outline of the planned workflow and processes that will be followed:

- Has a shared main repo (`upstream repository`) with two key branches: `main` and `dmz`
- Each contributor uses their own fork as their workspace

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sitecore/xmcloud-starter-js](https://github.com/Sitecore/xmcloud-starter-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
