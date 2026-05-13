---
trigger: always_on
description: > Everything an engineer or AI agent needs to understand, navigate, and contribute to this repository.
---

# AGENTS.md — ColdBru Repository Guide

> Everything an engineer or AI agent needs to understand, navigate, and contribute to this repository.

## Reply Guideline

- Always start your reply with ✨

## What Is ColdBru?

ColdBru is an independent fork of [Bruno](https://github.com/usebruno/bruno) — a local-first, offline-friendly API client. ColdBru adds opinionated UX on top: better navigation, a VS Code-style activity bar, Git tooling built into the sidebar, improved global search, and workspace polish. Collections are plain-text files on disk, not cloud-synced.

GitHub: <https://github.com/nicandcranny/coldbru>

## Repository Structure

```
coldbru/
├── packages/
│   ├── coldbru-app/          # React (Rsbuild) web UI — the renderer process
│   │   ├── src/
│   │   │   ├── components/   # React components (styled-components + Tailwind)
│   │   │   ├── hooks/        # Custom React hooks
│   │   │   ├── providers/    # React context providers
│   │   │   ├── selectors/    # Redux selectors
│   │   │   ├── utils/        # Shared utilities
│   │   │   ├── pages/        # Page-level components
│   │   │   ├── themes/       # Theme definitions
│   │   │   ├── styles/       # Global styles
│   │   │   ├── ui/           # Reusable UI primitives
│   │   │   ├── i18n/         # Internationalization
│   │   │   └── assets/       # Static assets
│   │   ├── storybook/        # Storybook config
│   │   └── jest/             # Jest transformers
│   └── coldbru-electron/     # Electron main process
│       ├── src/
│       │   ├── index.js      # Main process entry
│       │   ├── ipc/          # IPC handlers (main ↔ renderer)
│       │   ├── app/          # App lifecycle
│       │   ├── store/        # Electron-store persistence
│       │   ├── utils/        # Main-process utilities
│       │   ├── cache/        # Caching layer
│       │   ├── about/        # About window
│       │   └── preload.js    # Preload script
│       ├── tests/            # Jest tests for main process
│       ├── resources/        # Icons, entitlements, data files
│       └── web/              # Built web output served by Electron
├── scripts/                  # Dev, build, and release scripts
│   ├── dev.js                # `npm run dev` — starts web + electron
│   ├── dev-hot-reload.js     # `npm run dev:watch` — with hot reload
│   ├── release/
│   │   ├── build-electron.js # JS build orchestrator
│   │   ├── build-electron.sh # Shell wrapper for platform builds
│   │   ├── build-electron-linux-docker.sh # Linux x64 build on macOS via Docker
│   │   ├── collect-release-artifacts.js # Copies public artifacts into build/v<version>
│   │   └── assemble-release.js # Assembles release artifacts + SHA256SUMS
│   ├── generate-icons.js     # Icon generation
│   ├── setup.js              # First-time setup
│   ├── count-locs.js         # LOC counter
│   ├── pr-checkout.js        # PR checkout helper
│   └── changed-packages.js   # Detect changed packages
├── .github/
│   ├── workflows/
│   │   ├── tests.yml         # Unit tests CI (push + PR to main)
│   │   └── lint-checks.yml   # ESLint CI (push + PR to main)
│   ├── actions/              # Composite actions (setup-node-deps, run-unit-tests)
│   ├── PULL_REQUEST_TEMPLATE.md
│   ├── ISSUE_TEMPLATE/       # Bug report + feature request templates
│   ├── CODEOWNERS            # @nicandcranny
│   └── scripts/              # Flaky test detection
├── assets/                   # Logo, demo video
├── eslint.config.js          # Flat ESLint config
├── package.json              # Root workspace config
└── .nvmrc                    # Node v22.12.0
```

### Key Upstream Dependencies

ColdBru uses published `@usebruno/*` packages for core functionality:
- `@usebruno/common`, `@usebruno/converters`, `@usebruno/schema`, `@usebruno/lang`, `@usebruno/js`, `@usebruno/filestore`, `@usebruno/requests`

ColdBru-specific code lives in `packages/coldbru-app` and `packages/coldbru-electron`.

## Tech Stack

| Layer | Technology |
|---|---|
| UI framework | React 19, Redux Toolkit, styled-components, Tailwind CSS |
| Build (web) | Rsbuild |
| Desktop shell | Electron 41 |
| Desktop build | electron-builder |
| Testing | Jest, @testing-library/react |
| Linting | ESLint 9 (flat config), @stylistic/eslint-plugin |
| Pre-commit | Husky + nano-staged |
| Git operations | simple-git |
| Node version | v22.12.0 (see `.nvmrc`) |

## Getting Started

```bash
# use correct Node version
nvm use

# install dependencies
npm i --legacy-peer-deps

# start dev (web + electron together)
npm run dev

# or start separately
npm run dev:web      # terminal 1
npm run dev:electron # terminal 2

# hot-reload mode
npm run dev:watch
```

To isolate Electron app data during development:
```bash
ELECTRON_USER_DATA_PATH=$(realpath ~/Desktop/coldbru-test) npm run dev:electron
```

## Coding Guidelines

Full standards are in [CODING_STANDARDS.md](CODING_STANDARDS.md) ([GitHub](https://github.com/nicandcranny/coldbru/blob/main/CODING_STANDARDS.md)).

Summary of the most important rules:

- 2-space indentation, single quotes, semicolons always.
- No trailing commas. Always parenthesise arrow function params.
- Styled-components for colors (via theme prop, not CSS variables). Tailwind for layout only.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nicandcranny/coldbru](https://github.com/nicandcranny/coldbru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
