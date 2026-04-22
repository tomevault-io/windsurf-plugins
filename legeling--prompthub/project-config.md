---
trigger: always_on
description: **PromptHub** is a local-first, cross-platform desktop application for managing AI prompts. It allows users to organize, version control, and test prompts across multiple AI models. It also includes a **Skill system** for managing reusable AI skill definitions (SKILL.md files with frontmatter metadata).
---

# PromptHub — Project Context & Development Rules

## 1. Project Overview

**PromptHub** is a local-first, cross-platform desktop application for managing AI prompts. It allows users to organize, version control, and test prompts across multiple AI models. It also includes a **Skill system** for managing reusable AI skill definitions (SKILL.md files with frontmatter metadata).

- **Type:** Desktop Application (Electron)
- **License:** AGPL-3.0
- **Version:** 0.4.9

### Tech Stack

| Category            | Technology                                                         |
| :------------------ | :----------------------------------------------------------------- |
| **Runtime**         | Electron 33                                                        |
| **Frontend**        | React 18, TypeScript 5, Vite 6                                     |
| **Styling**         | Tailwind CSS 3 (design tokens: `bg-card`, `text-muted-foreground`) |
| **Icons**           | Lucide React                                                       |
| **State**           | Zustand 5                                                          |
| **Database**        | SQLite (via `better-sqlite3` in main process)                      |
| **Testing**         | Vitest 2 (unit), Playwright 1.57+ (E2E)                            |
| **I18n**            | i18next 24 / react-i18next 15 (7 locales)                          |
| **Package Manager** | pnpm                                                               |

## 2. Architecture

The application follows the standard Electron process model:

### Main Process (`src/main`)

- Handles native OS interactions, file system access, database operations, and security (encryption).
- **Entry:** `src/main/index.ts`
- **Database:** Direct SQLite access via `better-sqlite3`. Schema defined in `src/main/database/schema.ts`. Adapter in `src/main/database/sqlite.ts`.
- **IPC:** Exposes functionality to the renderer via `ipcMain.handle()` handlers registered in `src/main/ipc/`.
- **Security:** Master password, encryption/decryption in `src/main/security.ts`.
- **Services:** Business logic for skills (installer, validator, repo sync, sanitizer) in `src/main/services/`.

### Renderer Process (`src/renderer`)

- A React SPA responsible for the UI/UX.
- **Entry:** `src/renderer/main.tsx`
- **Communication:** Uses `window.api` (exposed via `contextBridge` in `src/preload/index.ts`) to call Main process methods.
- **State:** Zustand stores in `src/renderer/stores/` (prompt, folder, skill, settings, ui).
- **Services:** Frontend services in `src/renderer/services/` (AI client, WebDAV, skill platform sync, etc.).

### Data Layer

- **Local Storage:** SQLite database stores prompts, versions, folders, skills, skill versions, and settings.
- **Search:** Uses SQLite FTS5 for full-text search (`prompts_fts` virtual table).
- **Sync:** WebDAV support for backup and sync.
- **Skill Files:** Skills stored as SKILL.md files with YAML frontmatter metadata, synced between DB and local repo via `skill-repo-sync.ts`.

## 3. Key Commands

| Command                     | Description                            |
| :-------------------------- | :------------------------------------- |
| `pnpm install`              | Install dependencies                   |
| `pnpm electron:dev`         | Start dev server (Vite + Electron)     |
| `pnpm build`                | Build for production (Main + Renderer) |
| `pnpm electron:build`       | Build and package the application      |
| `pnpm test -- --run`        | Run full unit test suite (Vitest)      |
| `pnpm test -- <path> --run` | Run single test file                   |
| `pnpm test:e2e`             | Run end-to-end tests (Playwright)      |
| `pnpm lint`                 | Run ESLint                             |
| `pnpm format`               | Format code with Prettier              |

## 4. Directory Structure

```text
PromptHub/
├── src/
│   ├── main/                       # Electron Main Process
│   │   ├── database/               # SQLite schema, adapters, DB classes
│   │   │   ├── schema.ts           # DDL definitions (tables + indexes)
│   │   │   ├── sqlite.ts           # DatabaseAdapter (better-sqlite3 wrapper)
│   │   │   ├── index.ts            # DB initialization + schema migrations
│   │   │   ├── prompt.ts           # PromptDB (CRUD, versioning, FTS)
│   │   │   ├── folder.ts           # FolderDB (CRUD, reorder, hierarchy)
│   │   │   └── skill.ts            # SkillDB (CRUD, versioning)
│   │   ├── ipc/                    # IPC handler implementations
│   │   │   ├── index.ts            # Handler registration
│   │   │   ├── prompt.ipc.ts       # Prompt IPC handlers
│   │   │   ├── folder.ipc.ts       # Folder IPC handlers
│   │   │   ├── skill.ipc.ts        # Skill IPC entry (delegates to skill/)
│   │   │   ├── skill/              # Skill IPC sub-handlers
│   │   │   │   ├── crud-handlers.ts
│   │   │   │   ├── local-repo-handlers.ts
│   │   │   │   ├── platform-handlers.ts
│   │   │   │   ├── version-handlers.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [legeling/PromptHub](https://github.com/legeling/PromptHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
