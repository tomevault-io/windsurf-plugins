---
trigger: always_on
description: This file provides a comprehensive overview of the `lareview` project, intended to be used as instructional context for Gemini.
---

# GEMINI.md

This file provides a comprehensive overview of the `lareview` project, intended to be used as instructional context for Gemini.

## Project Overview

**LaReview** is a desktop code review companion that generates intent-driven review plans with AI. Built with Rust and Tauri, it provides a graphical user interface to review code changes (diffs), generate review tasks, and add notes. The application uses an external AI agent for parts of the review process, communicating via the Agent Client Protocol (ACP).

### Key Technologies

#### Backend (Rust)

| Technology                                                                | Purpose                 |
| ------------------------------------------------------------------------- | ----------------------- |
| [Rust](https://www.rust-lang.org/) (2024 Edition)                         | Core language           |
| [Tauri](https://tauri.app/) v2                                            | Desktop app shell & IPC |
| [`tokio`](https://tokio.rs/)                                              | Asynchronous runtime    |
| [`rusqlite`](https://github.com/rusqlite/rusqlite)                        | Local SQLite database   |
| [`handlebars`](https://crates.io/crates/handlebars)                       | Prompt templating       |
| [`agent-client-protocol`](https://crates.io/crates/agent-client-protocol) | AI agent communication  |
| [`pmcp`](https://crates.io/crates/pmcp)                                   | MCP client support      |
| [`syntect`](https://crates.io/crates/syntect)                             | Syntax highlighting     |

#### Frontend (React)

| Technology                                                  | Purpose                 |
| ----------------------------------------------------------- | ----------------------- |
| [React](https://react.dev/) v19                             | UI framework            |
| [TypeScript](https://www.typescriptlang.org/)               | Type-safe JavaScript    |
| [Tailwind CSS](https://tailwindcss.com/) v4                 | Styling                 |
| [Vite](https://vitejs.dev/) v7                              | Build tool              |
| [Zustand](https://zustand.docs.pmnd.rs/)                    | State management        |
| [Monaco Editor](https://microsoft.github.io/monaco-editor/) | Code diff viewer        |
| [TanStack Query](https://tanstack.com/query)                | Data fetching & caching |
| [Phosphor Icons](https://phosphoricons.com/)                | Icon library            |
| [Vitest](https://vitest.dev/)                               | Testing framework       |

---

## Architecture

The project follows a modular, layered architecture:

- **Domain layer** (`src/domain/`): Pure business logic, no external dependencies
- **Infrastructure layer** (`src/infra/`): External integrations (DB, VCS, ACP)
- **Commands** (`src/commands/`): Thin wrappers exposing functionality to frontend
- **Components** (`frontend/src/components/`): Organized by feature/view

---

## UI Views

LaReview has a Tauri-based desktop shell with three main views:

| View         | Description                                                                                      |
| ------------ | ------------------------------------------------------------------------------------------------ |
| **Generate** | Paste a diff (or fetch from GitHub PR), select an AI agent, and generate a review plan           |
| **Review**   | Tree-based navigation through the review plan; view task details, code changes, and add feedback |
| **Settings** | Configure AI agents, editor preferences, and application settings                                |

---

## Building and Running

### Prerequisites

- Rust nightly toolchain (see `rust-toolchain.toml`)
- Node.js & pnpm
- Tauri CLI: `cargo install tauri-cli`

### Development Commands

```bash
# Run the full application (backend + frontend)
cargo tauri dev

# Or run frontend dev server separately
cd frontend && pnpm dev
```

### Build Commands

```bash
# Build Rust backend
cargo build

# Build frontend
cd frontend && pnpm build

# Build production release
cargo tauri build
```

### Code Quality

```bash
# Check formatting
cargo fmt -- --check

# Run linter (warnings = errors)
cargo clippy --all-targets --all-features -- -D warnings

# Run Rust tests
cargo test

# Run frontend linter
cd frontend && pnpm lint

# Run frontend tests
cd frontend && pnpm test
```

---

## Development Conventions

### Formatting & Linting

- **Rust:** Uses `rustfmt` for formatting and `clippy` with strict policy (`-D warnings`)
- **Frontend:** Uses ESLint + Prettier with Tailwind plugin
- **Toolchain:** Nightly Rust as defined in `rust-toolchain.toml`

### Git Workflow

1. **Always** run `cargo fmt` and `cargo clippy` before committing
2. Use `git rebase` instead of `git merge` to keep history clean
3. Follow [Conventional Commits](https://www.conventionalcommits.org/) format: `action(scope): subject`
   - Examples: `feat(review): add feedback threading`, `fix(diff): handle binary files`
4. Write concise, descriptive commit messages that summarize all changes
5. Do not commit or push asking for review or approval

### Code Organization


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [puemos/lareview](https://github.com/puemos/lareview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
