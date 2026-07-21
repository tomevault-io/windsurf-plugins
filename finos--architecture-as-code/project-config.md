---
trigger: always_on
description: **IMPORTANT:** Use the `/sandbox/` folder for all temporary working files, test outputs, notes, and drafts.
---

# FINOS CALM Monorepo - AI Assistant Guide

## Sandbox Folder for Working Files

**IMPORTANT:** Use the `/sandbox/` folder for all temporary working files, test outputs, notes, and drafts.

- The `sandbox/` folder is in `.gitignore` and will not be committed
- Store test plans, results, TODO lists, and exploration notes here
- Do NOT create working files in other directories (they may accidentally be committed)
- Clean up the sandbox when work is complete if appropriate

```bash
# Example: Create working files in sandbox
mkdir -p sandbox
echo "My notes" > sandbox/my-notes.md
```

---

## Project Overview

This is the **FINOS Architecture as Code** monorepo containing the Common Architecture Language Model (CALM) specification and associated tools.

**CALM** is a declarative, JSON-based modeling language for describing complex software architectures, particularly in regulated environments like financial services.

## Monorepo Structure

```
architecture-as-code/
├── calm/                      # CALM specification (JSON schemas)
├── cli/                       # TypeScript CLI (@finos/calm-cli)
├── calm-hub/                  # Java/Quarkus REST API backend
├── calm-hub-ui/               # React frontend for CALM Hub
├── calm-server/               # TypeScript server (@finos/calm-server)
├── calm-plugins/vscode/       # VSCode extension
├── calm-models/               # TypeScript data models
├── calm-widgets/              # React visualization components
├── calm-ai/                   # AI agent tools & prompts
├── calm-suite/                # Sub-monorepos (see below)
│   ├── calm-studio/           #   SvelteKit visual CALM editor — nested npm-workspace monorepo
│   └── calm-guard/            #   Next.js continuous-compliance platform (CALMGuard)
├── shared/                    # Shared TypeScript utilities
├── docs/                      # Docusaurus documentation site
├── advent-of-calm/            # Educational content (24-day challenge)
├── experimental/              # Experimental features
├── template-bundles/          # Reusable Handlebars template bundles
├── conferences/               # Conference/workshop material
├── brand/                     # Logo and brand assets
└── scripts/                   # Repo maintenance scripts (e.g. lockfile validation)
```

### `calm-suite/` — nested workspaces

`calm-suite/` holds two products whose packages are wired directly into the **root** npm workspaces (run all npm commands from the repo root, never from inside these folders):

- **`calm-studio/`** (`calmstudio-workspace`) — a SvelteKit (Svelte 5) visual CALM editor, itself an npm-workspace monorepo. Sub-packages and the app are root workspaces via `calm-studio/packages/*` and `calm-studio/apps/*`: `@calmstudio/calm-core`, `@calmstudio/calmscript`, `@calmstudio/extensions`, `@calmstudio/github-action`, `@calmstudio/mcp`, `@calmstudio/diagram` (web-component), `calmstudio` (vscode-extension), and `@calmstudio/studio` (app).
- **`calm-guard/`** (`calmguard`) — a Next.js (App Router) continuous-compliance platform, plus its Docusaurus docs (`calmguard-docs`). Both are root workspaces.

## Technology Stack

**TypeScript/Node.js** (npm workspaces):
- CLI, models, widgets, shared, VSCode plugin, Hub UI, calm-server, calm-ai, and the `calm-suite/` products (CalmStudio's 8 packages/app + CALMGuard)
- Build: tsup (esbuild), vitest for testing
- Package manager: npm workspaces (single root lockfile; see [Lockfile Regeneration](#lockfile-regeneration))

**Java/Maven** (Maven reactor build):
- Root pom.xml defines multi-module reactor
- Modules: calm-hub (Java/Quarkus), cli, calm, docs, shared (POM modules)
- calm-hub backend (Quarkus 3.34+)
- MongoDB/NitriteDB storage
- TestContainers for integration tests
- Maven reactor allows building all modules from root: `./mvnw clean install`

**Documentation**:
- Docusaurus for main docs (and for CALMGuard's `calmguard-docs`)
- Astro for advent-of-calm website

## Node Version Requirements

**Canonical Node version: 26.** All CI workflows run on Node 26 via `node-version-file: '.nvmrc'`
(which pins `26.3.1`). The `engines` field (`>=26.0.0`) requires Node 26+, and `engine-strict`
in `.npmrc` blocks installs on anything older. **Node 26 is the version used to validate builds
and tests in CI.**

**Before running any commands**, verify your Node version:
```bash
node --version   # MUST show v26.x.x
```

If you are on the wrong version:
```bash
# If using nvm:
nvm use   # reads .nvmrc → 26.3.1
```

### Node 26 — Web Storage API and localStorage

Node 25+ shipped the [Web Storage API](https://nodejs.org/api/globals.html) as a global, meaning
`localStorage` and `sessionStorage` are now real globals in the Node runtime. Node 26 made this
stricter: accessing `localStorage` without `--localstorage-file` **throws a `DOMException`** at
runtime.

In Vitest's jsdom environment, Node's global **shadows** jsdom's `localStorage`, breaking any test
that calls methods like `.clear()` or `.getItem()`.

**How calm-hub-ui handles this:**

1. **Storage Dependency Injection** — `node-position-service.tsx`, `TimelineBar.tsx`, and
   `viewportStore.ts` all accept an optional `Storage` parameter (defaulting to `localStorage` /

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [finos/architecture-as-code](https://github.com/finos/architecture-as-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
