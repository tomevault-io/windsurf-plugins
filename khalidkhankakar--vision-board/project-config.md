---
trigger: always_on
description: A **real-time collaborative drawing application** (similar to Figma) built with Next.js, React 19, Liveblocks, Drizzle ORM, and PostgreSQL. See [README.md](README.md) for feature overview.
---

# Vision Board - Agent Instructions

A **real-time collaborative drawing application** (similar to Figma) built with Next.js, React 19, Liveblocks, Drizzle ORM, and PostgreSQL. See [README.md](README.md) for feature overview.

## 🎯 Project Status

**Stage**: MVP - mostly feature-complete, with some TODOs
- ✅ Authentication, real-time sync, board management, search, organization structure
- ⏳ **Incomplete**: Pencil tool drawing, toast notifications, full layer renderers, dynamic user IDs

## 🏗️ Architecture

### Directory Structure
```
app/
├─ (auth)              # Sign-in/Sign-up (Clerk)
├─ (dashboard)         # Dashboard with board list
│  └─ _components/     # 16 UI components (board-*, org-*, navbar, search)
├─ board/[id]          # Canvas page (protected)
│  └─ _components/     # 14 canvas components (drawing, layers, toolbar)
├─ api/[...route]/     # Hono.js API router
└─ layout.tsx          # Root layout

lib/
├─ db/                 # Drizzle ORM setup, schemas (board, favorite, user), migrations
├─ query/              # API call wrappers (board.queries.ts uses SWR + fetcher)
└─ utils.ts            # Geometry helpers, color utilities, bounds calculation

components/
├─ shared/             # room.tsx (Liveblocks provider), modals
└─ ui/                 # Radix/Shadcn components (button, dialog, avatar, etc)

hooks/                 # use-debounce.ts, use-delete-layer.tsx, use-rename-model.tsx, use-selection-bounds.tsx

providers/             # model-provider.tsx (Zustand modal state)

types/                 # canvas.ts (LayerType, canvasElements, interfaces)
```

### Data Flow
```
Client (SWR)
   ↓
Next.js API Route (Hono)
   ↓
Drizzle ORM + PostgreSQL (board, favorite)
   
Real-time Layer:
Client ←→ Liveblocks ←→ Other clients
(Canvas state synced via RoomProvider)
```

## 🛠️ Tech Stack & Key Dependencies

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Frontend** | Next.js 15, React 19 | App Router, Server Components |
| **UI** | Tailwind CSS, Radix UI, Shadcn | Styling & components |
| **State** | Zustand | Modal state (rename dialog) |
| **Data Fetch** | SWR | Client-side caching |
| **Real-time** | Liveblocks | Presence, awareness, storage |
| **Backend** | Hono.js | Lightweight API in route handlers |
| **Database** | Drizzle ORM, PostgreSQL (Neon) | Type-safe queries |
| **Auth** | Clerk | User authentication |

## 🚀 Development Workflow

### Essential Commands
```bash
npm run dev              # Start Next.js + Turbopack
npm run build           # Production build
npm run lint            # ESLint check
npm run db:generate     # Generate Drizzle migrations
npm run db:push         # Apply migrations to database
```

### Environment Variables (required)
```
DATABASE_URL            # PostgreSQL connection string
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY
CLERK_SECRET_KEY
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
LIVEBLOCKS_PUBLIC_API_KEY
```

## 📋 Key Conventions & Patterns

### File Naming
- Components: `kebab-case.tsx`
- Hooks: `use-kebab-case.ts(x)`
- Database: lowercase table names, camelCase columns
- Database indexes: `{column}Idx` suffix

### Component Patterns
```tsx
// Standard export + static skeleton for loading states
const BoardCard = ({ id, title, ... }) => { ... }
BoardCard.Skeleton = () => <div className="skeleton">...</div>
export default BoardCard
```

### Data Fetching (SWR + SWR)
```tsx
// board.queries.ts: wrapper functions
export const fetcher = (url) => fetch(url).then(r => r.json())

// Usage in components:
const { data, error, isLoading } = useSWR(endpoint, fetcher)
```

### Type Definitions
- Database: `IBoard`, `IFavorite` (from Drizzle schema)
- Canvas: `LayerType` enum (rectangle, ellipse, text, sticky_note) in [types/canvas.ts](types/canvas.ts)

### API Routes (Hono)
All routes in [app/api/[...route]/route.ts](app/api/[...route]/route.ts):
- `GET /api/board/:orgId` — List boards with search/favorite filter
- `POST /api/board` — Create board
- `PATCH /api/board/:id` — Update (title)
- `DELETE /api/board/:id` — Delete
- `PATCH /api/favorite/:boardId` — Toggle favorite
- `GET /api/organization/:orgId/members` — Get org members

## 🎨 Canvas & Real-time Layer

### Key Files
- [app/board/_components/canvas.tsx](app/board/_components/canvas.tsx) — 300+ lines: drawing logic, selection, mutations
- [components/shared/room.tsx](components/shared/room.tsx) — Liveblocks RoomProvider setup
- [liveblocks.config.ts](liveblocks.config.ts) — Type definitions for Liveblocks storage
- [types/canvas.ts](types/canvas.ts) — Canvas types (LayerType, Bounds, etc)

### Canvas State (Liveblocks)
- Storage: `layers` (map of layer objects), `layerIds` (order)
- Presence: `user` (cursor position, selectedLayerIds, pencilDraft)

### Drawing Operations
- **Selection**: Click shape → set `selectedLayerIds`
- **Move**: Drag selected layer → update layer position
- **Resize**: Handle resize → recalculate bounds
- **Undo/Redo**: Integrated with Liveblocks history
- **Delete**: useDeleteLayer hook mutation

### Layer Renderers
- [layer-preview.tsx](app/board/_components/layer-preview.tsx) — Renders individual layers by type

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [khalidkhankakar/vision-board](https://github.com/khalidkhankakar/vision-board) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
