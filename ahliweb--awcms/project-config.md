---
trigger: always_on
description: AWCMS is architected to be "AI-Native", meaning the codebase structure, naming conventions, and documentation are optimized for collaboration with AI Coding Assistants (Agents) like GitHub Copilot, Cursor, Claude, and Gemini.
---

# AI Agents Documentation

AWCMS is architected to be "AI-Native", meaning the codebase structure, naming conventions, and documentation are optimized for collaboration with AI Coding Assistants (Agents) like GitHub Copilot, Cursor, Claude, and Gemini.

## Documentation Authority

All agent work must respect this chain:

1. `SYSTEM_MODEL.md` (primary source of truth)
2. `AGENTS.md` (this file)
3. `README.md` (canonical monorepo operational baseline)
4. `DOCS_INDEX.md` (documentation routing)
5. implementation/module docs in `docs/**`

---

## 🤖 Agent Overview

In the AWCMS ecosystem, AI Agents are treated as specialized team members. We define three primary personas for AI interactions:

### 1. The Coding Agent (Architect/Builder)

- **Focus**: Implementation, Refactoring, Bug Fixing.
- **Capabilities**:
  - Full context awareness of React 19/Vite 8/Supabase constraints.
  - Ability to generate complex UI components using `shadcn/ui` patterns.
  - Writing SQL migrations for Supabase.
  - Updating system hooks (e.g., `useSearch`, `useAdminMenu`, `useMedia`, `useTwoFactor`).
- **Responsibility**: Ensuring code quality, functional patterns, and adhering to the "Single Source of Truth" principle.

### 2. The Communication Agent (Documenter/Explainer)

- **Focus**: Documentation, Changelogs, PR Descriptions.
- **Capabilities**:
  - Summarizing technical changes for non-technical stakeholders.
  - Updating Markdown files in `docs/` folder.
  - Generating "How-to" guides based on code analysis.
- **Responsibility**: Maintaining the accuracy of documentation relative to the codebase state.

### 3. The Public Experience Agent (Frontend Specialist)

- **Focus**: Public Portal (`awcms-public`), Astro Islands, Performance.
- **Capabilities**:
  - Working with **Astro 6.1.4** in `awcms-public/primary`, **Astro 6.0.8** in `awcms-public/smandapbun`, and **React 19.2.4** (Static output + Islands).
  - Implementing **Zod** schemas for component prop validation.
  - Optimizing for Cloudflare Pages static builds (cache headers, asset optimization).
- **Constraints**:
  - **NO** direct database access (must use Supabase JS Client or approved server-side edge runtimes).
  - **NO** Puck editor runtime in the public portal (use `Render` from `@puckeditor/core` only).

---

## 🔧 Current Tech Stack

Agents must be aware of the exact versions in use:

| Technology       | Version  | Notes                            |
| ---------------- | -------- | -------------------------------- |
| React            | 19.2.4   | Functional components only       |
| Vite             | `^8.0.5` | Build tool & dev server (`awcms`) |
| TailwindCSS      | `^4.2.2` / `^4.2.2` | Admin / Primary public |
| Supabase JS      | `^2.99.3` / `^2.99.3` | Admin / Primary public clients |
| React Router DOM | 7.10.1   | Client-side routing              |
| Puck             | 0.21.0   | Visual Editor (`@puckeditor/core`) |
| TipTap           | `^3.20.4` | Rich text editor (XSS-safe) — installed as `@tiptap/react`, `@tiptap/starter-kit`, `@tiptap/extension-image`, `@tiptap/extension-link`, `@tiptap/extension-placeholder`, `@tiptap/extension-underline`, `@tiptap/pm` |
| Framer Motion    | `^12.38.0` | Animations                     |
| Radix UI         | Latest   | Accessible UI primitives         |
| Lucide React     | 0.577.0  | Admin / Public icon library      |
| i18next          | `^25.10.3` | Internationalization           |
| Recharts         | 3.5.1    | Charts & Data Visualization      |
| Leaflet          | 1.9.4    | Maps                             |
| React Leaflet    | 5.0.0    | React bindings for Leaflet       |
| Vitest           | 4.1.0    | Unit/Integration testing         |
| Astro            | `6.1.4` / `6.0.8`  | Primary / SMANDAPBUN public portals |
| Wrangler         | `^4.77.0` | Cloudflare Worker CLI/dev server (`awcms-edge`) |

> [!IMPORTANT]
> **React Version Alignment**: The Admin Panel and Public Portal both use React 19.2.4. Ensure full compatibility with all dependencies.
> **Vite**: The admin workspace currently declares Vite `^8.0.5`; keep docs and implementation guidance aligned with the installed manifest version.
> **Node.js**: Minimum required version is **24.14.1** (standardized Node LTS baseline across AWCMS and OpenClaw). Managed via `nvm`.

---

## 📋 Agent Guidelines

To ensure successful code generation and integration, Agents must adhere to the following strict guidelines:

### Core Principles

1. **Context First**: Before generating code, read `README.md` and related component files to understand the existing patterns.

2. **Multi-Tenancy Awareness**:
   - **RLS is Sacred**: Never bypass RLS unless explicitly creating a Platform Admin feature (using `auth_is_admin()` or a server-side `SUPABASE_SECRET_KEY` path inside approved edge runtimes).
   - **Tenant Context**: Always use `useTenant()` or `usePermissions()` to get `tenantId`.
   - **Public Portal Tenant Context**: Static builds use `PUBLIC_TENANT_ID`/`VITE_PUBLIC_TENANT_ID`; avoid `Astro.locals` in build-time code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ahliweb/awcms](https://github.com/ahliweb/awcms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
