---
trigger: always_on
description: Leviathan is a modern, cross-platform Git GUI client built with **Tauri 2.0**, **Lit** (web components), and **Rust**. It's designed to be a fast, privacy-first alternative to existing Git clients with no telemetry, no account requirements, and offline-first operation.
---

# Copilot Instructions for Leviathan

## Project Overview

Leviathan is a modern, cross-platform Git GUI client built with **Tauri 2.0**, **Lit** (web components), and **Rust**. It's designed to be a fast, privacy-first alternative to existing Git clients with no telemetry, no account requirements, and offline-first operation.

### Key Technologies & Versions

- **Frontend**: TypeScript 5.3+, Lit 3.3+, Vite 7.3+
- **Backend**: Rust 1.70+, Tauri 2.0
- **Git Operations**: libgit2 (via git2-rs 0.20) for core operations, system Git for complex tasks
- **State Management**: Zustand 5.0+
- **Testing**: Web Test Runner, Playwright (E2E)
- **Build System**: Vite (frontend), Cargo (backend)

## Repository Structure

```
leviathan/
├── src/                      # Frontend (TypeScript/Lit)
│   ├── components/           # UI web components
│   │   ├── common/           # Shared components (buttons, inputs, etc.)
│   │   ├── dialogs/          # Modal dialogs
│   │   ├── graph/            # Commit graph visualization
│   │   ├── panels/           # Content panels (diff, blame, etc.)
│   │   ├── sidebar/          # Navigation sidebars
│   │   └── toolbar/          # Top toolbar
│   ├── services/             # API and service layer (Git, integrations)
│   ├── stores/               # Zustand state management
│   ├── types/                # TypeScript type definitions
│   ├── styles/               # CSS and design tokens
│   └── utils/                # Helper functions
│
├── src-tauri/                # Backend (Rust)
│   ├── src/
│   │   ├── commands/         # Tauri IPC command handlers
│   │   ├── services/         # Business logic
│   │   └── models/           # Data structures
│   ├── Cargo.toml            # Rust dependencies
│   └── tauri.conf.json       # Tauri configuration
│
├── e2e/                      # End-to-end tests (Playwright)
│   └── tests/                # E2E test specs
│
├── docs/                     # Additional documentation
├── .github/                  # GitHub configuration
│   ├── workflows/            # CI/CD pipelines
│   └── copilot-instructions.md
├── CLAUDE.md                 # Development guidelines for AI agents
└── README.md                 # Project documentation
```

## Installation & Setup

### Prerequisites

1. **Node.js** 20+ (for frontend development)
2. **Rust** 1.70+ (install via [rustup](https://rustup.rs/))
3. **System Git** 2.20+ (required for advanced Git operations)

### Platform-Specific Dependencies

#### macOS
```bash
xcode-select --install
```

#### Linux (Debian/Ubuntu)
```bash
sudo apt-get update
sudo apt-get install -y \
  libwebkit2gtk-4.1-dev \
  libayatana-appindicator3-dev \
  librsvg2-dev \
  libssl-dev \
  libgtk-3-dev \
  libglib2.0-dev \
  libdbus-1-dev
```

#### Windows
Install [Visual Studio Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/) with "Desktop development with C++" workload.

### Initial Setup

```bash
# Clone the repository
git clone https://github.com/hegsie/Leviathan.git
cd Leviathan

# Install frontend dependencies
npm install

# Start development mode (runs both frontend and backend)
npm run tauri:dev
```

## Development Workflow

### Build Commands

| Command | Description |
|---------|-------------|
| `npm run dev` | Start Vite dev server only (frontend) |
| `npm run tauri:dev` | Run app in development mode (frontend + backend) |
| `npm run tauri:build` | Build production application |
| `npm run build` | Build frontend only |

### Testing Commands

**IMPORTANT**: Always run tests before committing changes.

```bash
# Run all unit tests
npm test

# Run unit tests in watch mode (for TDD)
npm run test:watch

# Run end-to-end tests
npm run test:e2e

# Run E2E tests interactively
npm run test:e2e:ui

# Type check without emitting files
npm run typecheck
```

### Linting & Formatting

**CRITICAL**: These checks MUST pass before any commit.

```bash
# Lint TypeScript code
npm run lint

# Lint and auto-fix issues
npm run lint:fix

# Format code with Prettier
npm run format

# Check formatting without modifying files
npm run format:check

# Format Rust code
cd src-tauri && cargo fmt

# Lint Rust code with Clippy
cd src-tauri && cargo clippy
```

### Pre-commit Checklist

Run ALL of these before committing:

```bash
# Quick check (all-in-one)
npm run lint && npm run typecheck && cd src-tauri && cargo fmt --check && cargo clippy

# Also verify no snake_case in Tauri API calls (should return no matches)
grep -rn "_[a-z]*:" src/types/api.types.ts | grep -v "//"
```

## Coding Standards

### TypeScript/Frontend

1. **Web Components**: Use Lit web components with decorators
   - `@customElement('element-name')` for component registration
   - `@property()` for reactive properties
   - `@state()` for internal state

2. **Type Safety**:
   - Use explicit types, avoid `any` (warnings allowed with justification)
   - Enable strict mode TypeScript checks
   - Use TypeScript 5.3+ features

3. **Import Paths**: Use path aliases defined in `tsconfig.json`
   - `@/*` for `src/*`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hegsie/Leviathan](https://github.com/hegsie/Leviathan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
