---
trigger: always_on
description: Crumbs by Bretzel — self-hostable note-taking app. Multi-user with password auth + OAuth/SSO, offline-first PWA with CRDT-based sync.
---

# CLAUDE.md

## Project overview

Crumbs by Bretzel — self-hostable note-taking app. Multi-user with password auth + OAuth/SSO, offline-first PWA with CRDT-based sync.

## Tech stack

- **Framework**: SvelteKit with Svelte 5 (runes: `$state`, `$derived`, `$props`, `$effect`)
- **Adapter**: `@sveltejs/adapter-node` (builds to `build/`, runs via `node build`)
- **Styling**: Tailwind CSS v4 (via `@tailwindcss/vite`)
- **Database**: SQLite via better-sqlite3 + Drizzle ORM (WAL mode, schema at `src/lib/server/db/schema.ts`)
- **Auth**: Multi-user with password auth (Argon2) + OAuth (Google, GitHub, OIDC), role-based (admin/user), session cookies (30-day expiry)
- **PWA**: `@vite-pwa/sveltekit` for service worker and offline caching
- **Package manager**: pnpm

## Design system: Retro Parchment

The UI follows a **parchment + 8-bit retro** aesthetic. All design decisions must preserve this character.

### Color palette

All colors are CSS variables in `src/app.css`. Never use hardcoded Tailwind colors (e.g. `text-gray-500`, `bg-red-600`) — always reference CSS variables:

| Variable | Value | Usage |
|---|---|---|
| `--bg-base` | `#f0e6d3` | Page background (warm parchment) |
| `--bg-surface` | `#faf5eb` | Cards, panels, inputs |
| `--primary` | `#C8860A` | Brand gold — links, active states, accents |
| `--primary-hover` | `#E8A020` | Gold hover state |
| `--text` | `#1a1a2e` | Primary text (near-black) |
| `--text-muted` | `#6b6272` | Secondary text, placeholders |
| `--border` | `#1a1a2e` | Strong borders (editor, login card) |
| `--border-subtle` | `#d4cabb` | Default card borders, dividers |
| `--destructive` | `#a63d2f` | Delete actions, error icons |
| `--error-bg/border/text` | warm reds | Error states (themed, not clinical red) |
| `--success-bg/text` | warm greens | Success states |
| `--card-shadow` | `2px 2px 0px` | Hard-offset retro shadow (no blur) |
| `--card-shadow-hover` | `3px 3px 0px` | Hover shadow in primary gold |

Note colors for cards are defined in `src/lib/utils/colors.ts`.

### Typography

- **Body**: `JetBrains Mono` (monospace) — the hacker/dev aesthetic
- **Brand / empty states**: `Press Start 2P` (pixel font) — the 8-bit accent
- These two fonts are intentional. Do not replace them or add others.

### Visual rules

- **Shadows**: Always hard-offset (`2px 2px 0px`) with zero blur. Never use `shadow-sm`, `shadow-md` etc. — they look too modern.
- **Corners**: `rounded-sm` everywhere. Sharp, not rounded. Exception: color picker circles use `rounded-full`.
- **Borders**: Default card borders use `--border-subtle`. `--border` is for strong emphasis (editor, login card). Hover borders use `--primary`.
- **Animations**: Crisp and fast (150ms `ease-out`). No spring physics, no bounce. The retro feel comes from snappy transitions.
- **Background texture**: A subtle 4px pixel grid overlay at 3% opacity (defined in `body::before`).
- **Checkboxes**: Use `accent-color: var(--primary)` globally — gold checkboxes match the theme.
- **Icons**: Lucide icons throughout. Keep at 16-20px size.
- **Hover actions**: Never use `hidden group-hover:block` for action buttons — they're invisible on mobile (no hover). Use the opacity pattern instead: `max-md:opacity-100 md:opacity-0 transition-opacity md:group-hover:opacity-100`. This keeps actions always visible on touch devices and hover-revealed on desktop.

### Layout groups

- `(auth)` — Login/setup pages: minimal centered layout, no app chrome
- `(app)` — Main app: header, sidebar, footer, toast notifications

## Project structure

```
src/
  routes/
    +layout.svelte     # Root layout (imports CSS only)
    (auth)/            # Layout group: login/setup (minimal, no app chrome)
      +layout.svelte   # Centered parchment background
      login/           # Login page
      setup/           # First-time password setup
    (app)/             # Layout group: main app (header, sidebar, footer)
      +layout.svelte   # App shell with Header, Sidebar, Toast
      +page.svelte     # Main notes view
      archive/         # Archived notes view
      trash/           # Trashed notes view
      tag/[name]/      # Notes filtered by tag
      settings/        # Settings with tabbed layout (profile, MCP, users)
    api/auth/          # Login, logout, setup, OAuth endpoints
    api/admin/users/   # Admin user management (CRUD, role, sessions)
    api/notes/         # CRUD + attachments
    api/search/        # Full-text search
    api/sync/          # Offline sync
    api/tags/          # Tag management
  lib/
    components/        # Svelte components (NoteCard, NoteEditor, TagChip, etc.)
    stores/            # Svelte stores (notes.ts, theme.ts)
    server/db/         # Drizzle schema + connection (schema.ts, index.ts)
    server/            # Auth logic, attachment handling
    sync/              # Client sync (idb.ts), CRDT merge (crdt.ts), server sync
    types/             # TypeScript interfaces (Note, Attachment, Tag, etc.)
    utils/             # Colors, markdown, tags, debounce
  hooks.server.ts      # Auth middleware (redirects unauthenticated users)
tests/
  unit/                # Vitest unit tests
  e2e/                 # Playwright e2e tests (Gherkin-style)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bretzel-app/crumbs](https://github.com/bretzel-app/crumbs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
