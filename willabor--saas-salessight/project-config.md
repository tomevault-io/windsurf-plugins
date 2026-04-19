---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

**Excel Sales Data Processor** - A full-stack web application for processing, managing, and analyzing retail inventory and sales data from Excel files. Features include file upload/parsing, database storage, real-time statistics dashboard, data management with search/pagination, and upload history tracking.

## Development Commands

- **Development**: `npm run dev` - Starts the development server with Vite HMR and Express backend (port 5000)
- **Build**: `npm run build` - Builds the client (Vite) and server (esbuild) for production
- **Production**: `npm start` - Runs the production build
- **Type Checking**: `npm run check` - Runs TypeScript compiler to check types
- **Database Push**: `npm run db:push` - Pushes Drizzle schema changes to the database (no migrations folder - uses direct push)

## Architecture Overview

This is a full-stack TypeScript application with React frontend and Express backend, using a monorepo structure.

### Tech Stack

**Frontend:**
- React 18 with TypeScript (via Vite)
- Wouter for routing (lightweight, not React Router)
- TanStack Query (React Query) for server state management
- shadcn/ui component library (48 components, ~15 actively used)
- Tailwind CSS with custom design tokens
- SheetJS (xlsx) for Excel file processing
- React Hook Form + Zod validation

**Backend:**
- Node.js with Express.js
- Drizzle ORM with Neon serverless PostgreSQL
- express-session with memorystore (dev only - not production-ready)
- Zod validation (shared with frontend)

### Project Structure

```
client/
├── src/
│   ├── components/
│   │   ├── ui/                     # 48 shadcn/ui components
│   │   ├── excel-formatter.tsx    # Main upload/processing UI
│   │   ├── file-upload-tabs.tsx   # Tabbed upload interface
│   │   ├── stats-overview.tsx     # Dashboard metrics
│   │   ├── upload-progress.tsx    # Progress tracking
│   │   └── recent-activity.tsx    # Upload history
│   ├── pages/
│   │   ├── dashboard.tsx          # Main landing page
│   │   ├── item-list.tsx          # Item management (399 lines)
│   │   └── not-found.tsx          # 404 page
│   ├── lib/
│   │   ├── excel-processor.ts     # Excel parsing & validation
│   │   ├── formatters.ts          # Complex Excel formatting (760 lines!)
│   │   ├── api.ts                 # API client functions
│   │   ├── queryClient.ts         # React Query setup
│   │   └── utils.ts               # Utility functions
│   ├── hooks/                     # Custom React hooks
│   ├── App.tsx                    # Root component with routing
│   └── main.tsx                   # Entry point

server/
├── index.ts                       # Server entry point (71 lines)
├── routes.ts                      # 10 REST API endpoints (240 lines)
├── storage.ts                     # Database operations (238 lines)
├── db.ts                          # Drizzle connection (16 lines)
└── vite.ts                        # Vite dev middleware

shared/
└── schema.ts                      # Drizzle schemas + Zod validation (109 lines)
```

### Database Architecture

Uses Drizzle ORM with Neon serverless PostgreSQL (WebSocket-based connection):

**Tables:**
- `item_list` (22 fields) - Inventory items with quantities across 8 store locations, pricing, dates, metadata
- `sales_transactions` (9 fields) - Sales records with receipt info, SKU, pricing
- `upload_history` (8 fields) - Tracks file uploads with success/failure statistics
- `users` (3 fields) - User management stub (not actively used - no auth implemented)

**Key Details:**
- Schema location: `shared/schema.ts` (shared between client/server for type consistency)
- Storage interface: `server/storage.ts` exports `storage` singleton implementing `IStorage`
- All database operations go through the storage layer, never directly via `db`
- No migrations folder - using `drizzle-kit push` directly (⚠️ risky for production)

### Import Aliases

- `@/` → `client/src/`
- `@shared/` → `shared/`
- `@assets/` → `attached_assets/`

### Development Workflow

In development mode (`npm run dev`):
1. Express server starts on port 5000 (or PORT env var)
2. Vite middleware provides HMR for React app
3. API routes under `/api/*` handled by Express
4. All other routes serve the React SPA

In production mode (`npm start`):
1. Serves pre-built static files from `dist/public`
2. Express handles API routes
3. Single server process

### Data Upload Flow

**Item List Upload:**
1. User selects Excel file → `client/src/components/excel-formatter.tsx`
2. Client parses with SheetJS → `client/src/lib/excel-processor.ts`
3. Validation with Zod schemas → `shared/schema.ts`
4. API request → `POST /api/upload/item-list` → `server/routes.ts:85`
5. Storage layer → `server/storage.ts:73` (createItemList or upsertItemList)
6. Database insertion → `item_list` table via Drizzle ORM
7. Upload history recorded → `upload_history` table with stats
8. Response → Success/failure counts + first 5 errors
9. React Query invalidates cache → UI auto-refreshes

**Upload Modes:**
- `initial` - Inserts new records (fails on duplicates)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Willabor) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
