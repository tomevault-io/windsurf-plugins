---
trigger: always_on
description: > Guide for AI agents contributing to this codebase.
---

# AGENTS.md

> Guide for AI agents contributing to this codebase.

## Project Overview

**ระบบบริหารงาน Todo** — a full-featured Todo task management system built with React 19, TypeScript, Vite, and Tailwind CSS v4. The application provides Kanban boards, calendar views, dashboards with analytics, team management, audit logs, notifications, and an admin panel — all localized in Thai.

## Tech Stack

| Category       | Technology                          |
|----------------|-------------------------------------|
| Language       | TypeScript (~5.8)                   |
| Framework      | React 19                            |
| Build Tool     | Vite 6                              |
| Styling        | Tailwind CSS v4 (via `@tailwindcss/vite`) |
| Icons          | lucide-react                        |
| Charts         | recharts                            |
| Animations    | motion (Framer Motion)              |
| AI Integration | `@google/genai` (Gemini API)        |
| Backend API    | Express (server-side Gemini proxy)  |
| Type Checking  | `tsc --noEmit`                      |

## Path Aliases

- `@/*` → project root (`.`)

The Vite config maps `@` to the project root. Import using `@/` only; do not use relative `../` for cross-feature imports when `@/` is cleaner.

## File Structure

```
.
├── AGENTS.md                 # This file
├── package.json
├── vite.config.ts
├── tsconfig.json
├── index.html
├── metadata.json
├── .env.example
├── .gitignore
├── assets/
│   └── .aistudio/
├── src/
│   ├── main.tsx              # React entry point (createRoot)
│   ├── App.tsx               # Root component; renders Navbar, Sidebar, modals
│   ├── index.css             # Tailwind base + custom CSS variables
│   ├── types.ts              # All shared TypeScript types/interfaces
│   ├── data/
│   │   └── initialData.ts    # Seed/seed data (users, teams, tasks, logs, etc.)
│   ├── context/
│   │   └── AppContext.tsx    # Global state via React Context + useApp() hook
│   └── components/
│       ├── Navbar.tsx
│       ├── Sidebar.tsx
│       ├── TaskFormModal.tsx
│       ├── TaskDetailModal.tsx
│       ├── TaskListView.tsx
│       ├── TaskKanbanView.tsx
│       ├── TaskCalendarView.tsx
│       ├── DashboardView.tsx
│       ├── TeamManagementView.tsx
│       ├── TrashView.tsx
│       ├── AuditLogView.tsx
│       ├── AdminUsersView.tsx
│       ├── NotificationDrawer.tsx
│       └── ProfileModal.tsx
```

## Commands

All commands are run from the project root.

| Command         | Description                          |
|-----------------|--------------------------------------|
| `npm run dev`    | Start Vite dev server (port 3000)     |
| `npm run build`  | Production build via Vite             |
| `npm run preview`| Preview the production build locally  |
| `npm run lint`   | Run `tsc --noEmit` (type checking)    |
| `npm run clean`  | Remove `dist/` and `server.js`        |

**Always run `npm run lint` after changes** to ensure type correctness before finishing.

## Code Style & Conventions

### TypeScript

- Use `type` for type aliases (e.g., `UserRole`, `TaskStatus`, `TaskPriority`).
- Use `interface` for object/data shapes (e.g., `User`, `Task`, `Team`).
- Define all shared types in `src/types.ts`; import from there rather than redefining.
- Always annotate React components with `React.FC` (or `React.FC<PropsType>` for props).
- Use union types for enums (e.g., `status: 'not_started' | 'in_progress' | 'completed' | 'cancelled'`).
- Never use `any`; prefer `unknown` and narrow, or use discriminated unions.
- Use `Omit<>`, `Partial<>`, and `Pick<>` for derived types (e.g., `Omit<Task, 'id'>`).

### React / Component Patterns

- **State management:** Use the global `useApp()` hook from `AppContext.tsx` for shared state (tasks, users, teams, notifications, active view, etc.). Do not create new top-level contexts unless architecting a new domain.
- **Functional components only:** No class components.
- **Props:** Destructure props at the component signature.
- **Event handlers:** Name with `handle` prefix for complex logic, `on` prefix matching JSX prop names (e.g., `onToggleMobileSidebar`).
- **Modals:** Follow the existing controlled-modal pattern — a boolean state + `onClose` + `<Modal isOpen={...} onClose={...} />` structure driven from `AppContext` flags (`isTaskFormOpen`, `selectedTaskId`, etc.).
- **Views:** Each "view" component is a self-contained `React.FC` that reads from `useApp()`. Switching views is done via `setActiveView(string)`.
- **Local UI state** (form inputs, temporary UI toggles) lives in `useState` within the component; application data lives in `AppContext`.

### Imports

Order imports as follows:

1. External React/library imports (`react`, `react-dom`)
2. Third-party library imports (`lucide-react`, `recharts`, `motion`, etc.)
3. Type imports from `../types`
4. Context/hook imports (`../context/AppContext`)
5. Local component/data imports (`../components/*`, `../data/*`)

### Styling

- Use **Tailwind CSS utility classes**; no external CSS files or styled-components.
- The color/theme palette is defined via CSS custom properties in `src/index.css` under `:root`:
  - `--bg`, `--sidebar`, `--primary`, `--secondary`, `--border`, `--text-dark`, `--text-light`, `--danger`, `--warning`, `--success`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wanjantp/todo-next](https://github.com/wanjantp/todo-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
