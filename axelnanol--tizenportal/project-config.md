---
trigger: always_on
description: > **Last Updated:** February 21, 2026
---

# Copilot Instructions for TizenPortal

> **Last Updated:** February 21, 2026  
> **Current Version:** 1088  
> **Architecture:** Universal Runtime

---

## 🤖 How to Use These Instructions

This file provides comprehensive guidance for working on the TizenPortal codebase. As an AI coding agent:

1. **Read completely before starting** — Understanding context prevents mistakes
2. **Follow the constraints** — Chrome 47 compatibility is non-negotiable
3. **Make minimal changes** — Surgical edits only; don't refactor unnecessarily
4. **Test incrementally** — Build after every change
5. **Document as you go** — Update docs when behavior changes

### Critical Context

- **Target Platform:** Samsung Tizen TVs with Chrome 47-69 browser engine
- **Build System:** Rollup + Babel transpiling to ES5
- **Input Method:** TV remote only (D-pad + color buttons)
- **Testing:** Manual only (no automated tests)
- **Deployment:** Git tags → TizenBrew CDN

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Platform Constraints](#2-platform-constraints)
3. [Architecture](#3-architecture)
4. [Build System](#4-build-system)
5. [TizenBrew Integration](#5-tizenbrew-integration)
6. [Development Guidelines](#6-development-guidelines)
7. [Critical Constraints](#7-critical-constraints)
8. [Color Button Mappings](#8-color-button-mappings)
9. [Key Codes Reference](#9-key-codes-reference)
10. [Bundle System](#10-bundle-system)
11. [Deployment Workflow](#11-deployment-workflow)
12. [Common Pitfalls](#12-common-pitfalls)
13. [Agent Instructions](#13-agent-instructions)
14. [Attribution Requirements](#14-attribution-requirements)
15. [Documentation Standards](#15-documentation-standards)

---

## Quick Start for Copilot

### Essential Commands

```bash
# Install dependencies
npm install

# Build the project
npm run build

# Watch mode for development
npm run watch

# Clean build artifacts
npm run clean
```

### Testing
There is no automated test suite. Changes must be manually verified on a Samsung Tizen TV or through careful code review.

### Key Files to Know
- `core/index.js` — Main runtime entry point
- `core/utils.js` — Shared utilities (injectCSS, safeLocalStorageSet, log/warn)
- `bundles/registry.js` — Bundle registration system
- `bundles/registry.generated.js` — Auto-generated bundle registry (do not edit manually)
- `ui/portal.js` — Portal launcher UI
- `ui/preferences.js` — Preferences modal
- `ui/siteeditor.js` — Site card editor modal
- `ui/addressbar.js` — Address bar overlay (Red button)
- `input/handler.js` — Remote control key handling
- `features/registry.js` — Unified registry for features and userscripts
- `features/index.js` — Feature registrations
- `features/userscript-registry.js` — Userscript registrations
- `navigation/spatial-navigation.js` — Spatial navigation library

### Common Issues & Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| Build fails with "rollup: not found" | Dependencies not installed | Run `npm install` |
| "this is undefined" warning | Normal for UMD modules | Safe to ignore |
| Code works in modern browser but not on TV | Using ES6+ features | Check Babel transpilation |
| Changes not appearing on TV | CDN cache | Create new git tag |
| localStorage errors | Quota exceeded | Implement error handling |
| Focus lost after navigation | Missing spatial nav setup | Check focus groups |

---

## 1. Project Overview

**TizenPortal** is a browser shell for Samsung Tizen TVs that provides:

- **Portal Grid:** Launcher with customizable site cards
- **Bundle System:** Site-specific fixes and enhancements
- **Spatial Navigation:** TV-remote-friendly navigation
- **Diagnostics:** Debug overlay with console capture

### Mission

Provide TizenBrew users with a **usable browser shell** that works with contemporary SPAs (Audiobookshelf, Jellyfin, etc.) on legacy Tizen hardware where the built-in browser is Chrome 47-69.

---

## 2. Platform Constraints

### Target Hardware

- **Devices:** Samsung Tizen Smart TVs (2017-2022)
- **Browser Engine:** Chrome 47-69 (Chromium-based WebKit)
- **Resolution:** 1920×1080 fixed (never use responsive layout)
- **Input:** Samsung remote (directional pad, color buttons, media keys)

### Polyfill Philosophy

**DO NOT target specific browser versions.** Instead:
- Use **feature detection** for all capability decisions
- Load polyfills **only when needed**
- Assume **V8 only** (no other JS engines)
- Avoid heavy polyfills (no full core-js)

### JavaScript Considerations

```js
// ❌ NOT SUPPORTED in Chrome 47
const { a, b } = obj;           // Destructuring (limited)
[...arr]                        // Spread operator (limited)
async/await                     // Not available
fetch()                         // Not available (use XMLHttpRequest)
class Foo {}                    // Limited class syntax
() => {}                        // Arrow functions (use Babel)
for (const x of arr)            // for...of (limited)
Object.entries/values           // Not available
Array.includes()                // Not available

// ✅ SAFE in Chrome 47 (or Babel transpiles)
var, let, const
function declarations
Array.prototype.forEach/map/filter/reduce
Object.keys()
JSON.parse/stringify
XMLHttpRequest
setTimeout/setInterval
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [axelnanol/tizenportal](https://github.com/axelnanol/tizenportal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
