---
trigger: always_on
description: This document is a user-level guide for AI assistants working in this repository. It standardizes language, tool/script preferences, and development notes for the current tech stack.
---

# User Guide

This document is a user-level guide for AI assistants working in this repository. It standardizes language, tool/script preferences, and development notes for the current tech stack.

## 💬 Communication Conventions

- **Language**: Use English consistently for conversation, TODOs, and code-related content (comments, UI copy, commit messages, PR descriptions, and similar artifacts).
- **Conclusion first**: Start with the core conclusion/summary, then provide details.
- **References**: When citing code, always provide full file paths (for example, `src/main.ts:42`).

## 💻 Runtime and Tooling

- **Runtime**: Node.js (Electron environment)
- **Node**: Recommended Node.js 20+
- **Package manager**: `npm` (this project includes `package-lock.json`; use npm only)
- **Build tools**: Electron Forge + Vite
- **Terminal**: Windows (PowerShell) / VSCode MCP tools can be used safely

## 🧩 Tech Stack Overview

- **Frontend**:
  - React 19, TypeScript
  - Tailwind CSS v4, `clsx`, `tailwind-merge`, `tailwindcss-animate`
  - Radix UI (Primitives), Lucide React (Icons), Simple Icons (`@icons-pack/react-simple-icons`)
  - `class-variance-authority` (CVA), `react-i18next` + `i18next`
  - TanStack Router (Routing), TanStack Query (State Management)
  - Components: Modular design under `src/components`
- **Backend (Electron Main/Server)**:
  - Electron (Main/Preload/Renderer architecture)
  - NestJS (internal proxy/gateway service, started by main process)
  - Better-SQLite3 (local database), Drizzle ORM / Raw SQL
  - ORPC (type-safe RPC)
  - gRPC (`@grpc/grpc-js`, `@grpc/proto-loader`)
  - Logging: `winston` + `winston-daily-rotate-file`
  - Zod (validation)
- **Testing**:
  - Vitest (unit/integration), Testing Library
  - Playwright (E2E)

## 📁 Directory Structure

```plaintext
.
├─ src/
│  ├─ actions/           # App actions and flow orchestration
│  ├─ assets/            # Static assets
│  ├─ components/        # React UI components (base components under ui/)
│  ├─ constants/         # Constants
│  ├─ hooks/             # Custom React hooks
│  ├─ ipc/               # Electron IPC logic (Database, Config, etc.)
│  ├─ layouts/           # Layout components
│  ├─ lib/               # Shared low-level utilities
│  ├─ localization/      # i18n translation resources
│  ├─ mocks/             # Mock data for tests and development
│  ├─ routes/            # TanStack Router route definitions
│  ├─ server/            # NestJS backend logic (Gateway/Proxy)
│  ├─ services/          # Service layer
│  ├─ styles/            # Global styles (Tailwind classes)
│  ├─ tests/             # Test code
│  ├─ types/             # TypeScript type definitions
│  ├─ utils/             # Utility functions
│  ├─ App.tsx            # React app entry
│  ├─ main.ts            # Electron main entry
│  ├─ preload.ts         # Electron preload script
│  └─ renderer.ts        # Electron renderer entry
├─ forge.config.ts       # Electron Forge config
└─ package.json
```

## 🧱 Component Architecture

- **Modular components**: Each component should have its own directory, with at least a `.tsx` file and optional styles/subcomponents.
- **Shared capabilities**: General helpers in `src/utils/`; low-level shared wrappers in `src/lib/`.
- **Service layer**: Centralize data access in `src/services/` or `src/ipc/`; frontend should consume IPC or RPC only.

## 📦 Common Scripts

Use `npm` for all commands:

- **Development (Dev)**:
  - `npm start` - Start Electron dev environment (Electron Forge)
  - `npm run lint` - Run ESLint checks
  - `npm run format` - Run Prettier check
  - `npm run format:write` - Auto-format with Prettier
  - `npm run type-check` - Run TypeScript type check

- **Build**:
  - `npm run package` - Package app (application bundle only)
  - `npm run make` - Build and generate distributable installers
  - `npm run publish` - Publish app

- **Testing**:
  - `npm test` - Run Vitest tests
  - `npm run test:watch` - Run Vitest in watch mode
  - `npm run test:unit` - Same as above for unit-focused runs
  - `npm run test:e2e` - Run Playwright E2E tests
  - `npm run test:all` - Run all tests

### Running a Single Test

- Unit test: `npm run test:unit path/to/test.test.ts`
- E2E test: `npm run test:e2e path/to/test.spec.ts`
- Type check: `npm run type-check`

## 🧪 Development Notes

- **Build**: Build stage may ignore TS/ESLint errors depending on project/CI configuration.
- **DevTools**: `code-inspector-plugin` is integrated; use `Shift + Click` on page elements to jump to source code.
- **React**: React Strict Mode is disabled.
- **NestJS**: Runs as an Electron child process; logs are visible in main-process console.

## Security and Data

- **Security**: Never commit secrets; use environment variables for sensitive config; validate all user input; encrypt sensitive data.
- **Database**: Use Better-SQLite3; encapsulate operations in services layer; always use prepared statements; test DB operations independently.
- **i18n**: Use `react-i18next`; keys should use kebab-case; translation files are stored in `src/localization/`.

## 📝 Conventions

- **File naming**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Draculabo/AntigravityManager](https://github.com/Draculabo/AntigravityManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
