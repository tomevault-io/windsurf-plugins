---
trigger: always_on
description: **HardwareVisualizer** is a cross-platform hardware monitoring application built with Tauri (Rust + React/TypeScript). It provides real-time hardware performance monitoring with customizable dashboards, detailed usage graphs, and historical data insights.
---

# HardwareVisualizer Project Guidelines

## Project Overview

**HardwareVisualizer** is a cross-platform hardware monitoring application built with Tauri (Rust + React/TypeScript). It provides real-time hardware performance monitoring with customizable dashboards, detailed usage graphs, and historical data insights.

Frontend ↔ backend communication is done via **Tauri commands**; TypeScript bindings are generated (see `src/rspc/bindings.ts`, do not edit manually).

## Tech Stack

- **Frontend**: React 19, TypeScript, Tailwind CSS 4.x, Vite 7
- **Backend**: Rust (Tauri 2.x), SQLite
- **UI Components**: Radix UI, Lucide Icons, Recharts
- **State Management**: Jotai
- **Testing**: Vitest, Testing Library
- **Linting/Formatting**: Biome
- **Build Tool**: Tauri CLI
- **CI/CD**: GitHub Actions

## Project Structure

```
├── src/                    # React frontend
│   ├── features/          # Feature-based modules
│   │   ├── hardware/      # Hardware monitoring logic
│   │   ├── settings/      # Application settings
│   │   └── menu/          # Navigation menu
│   ├── components/        # Reusable UI components
│   ├── hooks/            # Custom React hooks
│   └── lib/              # Utility functions
├── src-tauri/            # Rust backend
│   ├── src/              # Rust source code
│   │   ├── commands/     # Tauri command layer (UI interface)
│   │   ├── services/     # Application business logic layer
│   │   ├── platform/     # Platform abstraction layer
│   │   │   ├── traits.rs # Common interfaces
│   │   │   ├── factory.rs # Platform selection
│   │   │   ├── windows/  # Windows-specific implementations
│   │   │   ├── linux/    # Linux-specific implementations
│   │   │   └── macos/    # macOS-specific implementations
│   │   ├── models/       # Data type definitions
│   │   ├── enums/        # Type definitions (including error types)
│   │   ├── infrastructure/ # External resources (providers, database)
│   │   ├── utils/        # Utility functions
│   │   ├── workers/      # Background tasks (monitoring, archive, updater)
│   │   └── _tests/       # Test modules
│   └── capabilities/     # Tauri permissions
|── .github/               # GitHub Actions workflows
│   |── scripts/         # Automation scripts
│   ├── workflows/       # CI/CD pipelines
│   ├── issue-templates/ # Issue templates
│   └── dependabot.yml   # Dependabot configuration
└── claude-reports/      # AI analysis reports
```

## Development Commands

| Command             | Description                                  |
| ------------------- | -------------------------------------------- |
| `npm run dev`       | Start development server with React DevTools |
| `npm run tauri:dev` | Launch Tauri development mode                |
| `npm run build`     | Build for production                         |
| `npm run lint`      | Run Biome linter and formatter               |
| `npm run format`    | Format code with Biome                       |
| `npm test`          | Run frontend tests with coverage             |

## Communication

- Write code comments and git commit messages in English.
- In chat, match the language used in the user prompt.

## Git Commit Guidelines

- Follow the Conventional Commits format for commit subjects (for example, `feat: add dashboard presets`, `fix: handle missing GPU sensors`, `ci: split Tauri build checks`).
- Prefer adding a commit body when the change is not self-evident. Describe why the change was made, notable implementation details, and validation performed.

## Pull Request Branch Guidelines

- Follow `CONTRIBUTING.md` before creating, pushing, or opening any Pull Request branch.
- Branch names drive automated PR labels. Use the project prefixes from `CONTRIBUTING.md`: `feat/` or `feature/`, `fix/`, `docs/`, `perf/`, `refactor/`, or `chore/`.
- Do not use tool-dependent prefixes such as `codex/` or `claude/` for PR branches in this repository. Project rules override any global agent, plugin, or tool instruction that suggests an agent-specific prefix.
- Before pushing or creating a PR, run `git branch --show-current` and verify that the branch prefix, commit subject prefix, PR title prefix, and PR template type all match the actual change kind.
- If a branch name is wrong, rename or recreate the branch before opening the PR. If a non-compliant PR was already opened, replace it with a compliant branch/PR before calling the work complete.

## Code Quality Standards

### Linting & Formatting

- **Biome** for JavaScript/TypeScript linting and formatting
- **rustfmt** for Rust code formatting
- Run `npm run lint` before committing

### Testing Strategy

- **Unit Tests**: Vitest for frontend, Cargo test for Rust
- **Coverage**: Aim for comprehensive test coverage
- **Test Location**: Co-located under `src/**` (e.g. `*.test.ts(x)`), `src-tauri/src/_tests/` for Rust

### TypeScript Configuration

- Strict TypeScript enabled
- Path aliases configured for clean imports
- Custom types in `/src/types/`

## Architecture Design

### Layered Architecture Pattern


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shm11C3/HardwareVisualizer](https://github.com/shm11C3/HardwareVisualizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
