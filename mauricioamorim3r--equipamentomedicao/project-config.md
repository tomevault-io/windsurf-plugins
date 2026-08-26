---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Metrological Equipment Management System** for oil & gas operations, built with:
- **Frontend**: React + TypeScript + Vite + Wouter (routing) + TanStack Query + Radix UI + Tailwind CSS
- **Backend**: Express.js + TypeScript
- **Database**: PostgreSQL (Neon) with Drizzle ORM
- **Deployment**: Render.com (production)

The system manages calibrations, equipment, measurement points, orifice plates, wells testing, chemical analysis, seals/locks, uncertainty control, and notifications for oil & gas production facilities.

## Essential Commands

### Development
```bash
npm run dev              # Start development server (tsx server/index.ts)
npm run build            # Build for production (vite build + esbuild server)
npm start                # Run production build
npm run check            # TypeScript check (currently disabled - outputs message)
npm run check-strict     # Strict TypeScript check (tsc without noEmit)
```

### Database Management
```bash
npm run db:push          # Push schema changes to database (drizzle-kit push)
npm run db:migrate       # Run migrations (drizzle-kit migrate)
npm run db:generate      # Generate migrations from schema (drizzle-kit generate)
```

### Database Seeding & Scripts
- Scripts are located in `server/scripts/`
- Key scripts include:
  - `seed-data.ts` - Seeds initial data (polos, campos, instalações, equipment, etc.)
  - `inserir-placas-ajustadas.ts` - Insert/update orifice plates
  - `relatorio-status-final.ts` - Generate status reports
  - `verificar-*.ts` - Verification scripts for data integrity

To run scripts: `tsx server/scripts/<script-name>.ts`

## Architecture

### Monorepo Structure
```
client/              # React frontend (Vite root is here)
  src/
    components/      # Reusable UI components
    pages/           # Route pages (lazy loaded)
    lib/             # Utilities, query client, hooks
server/              # Express backend
  routes.ts          # API route definitions
  storage.ts         # Database abstraction layer (IStorage interface)
  db.ts              # Database connection (Drizzle + Neon)
  scripts/           # Database seed & utility scripts
  upload.ts          # File upload (multer)
  exportUtils.ts     # Excel/report exports
  templateUtils.ts   # Import/export templates
shared/              # Shared code between client/server
  schema.ts          # Drizzle schema & Zod validation
```

### Key Architectural Patterns

1. **Database Layer**: All database operations go through `storage.ts` which implements the `IStorage` interface. Never query the database directly elsewhere - use storage methods.

2. **Schema Definition**: `shared/schema.ts` is the single source of truth for:
   - Database tables (Drizzle)
   - Relations
   - Validation schemas (Zod)
   - TypeScript types (exported from table inference)

3. **API Routes**: All API routes are in `server/routes.ts`:
   - Health checks: `/api/health` and `/api/health/database`
   - REST patterns: GET/POST/PATCH/DELETE for all entities
   - Exports: `/api/export/*` endpoints
   - File uploads: Use multer middleware from `upload.ts`

4. **Frontend State**: Uses TanStack Query for server state:
   - Query client configured in `client/src/lib/queryClient.ts`
   - Custom hooks in `client/src/lib/` for API interactions
   - Mutations invalidate queries to refetch data

5. **Routing**: Wouter (not React Router) for client-side routing:
   - Routes defined in `client/src/App.tsx`
   - All pages are lazy-loaded for code splitting
   - Base path is `/` (not `/client`)

6. **Path Aliases**:
   - `@/*` → `client/src/*`
   - `@shared/*` → `shared/*`
   - `@assets/*` → `attached_assets/*` (configured in vite.config.ts and tsconfig.json)

### Critical Database Relationships

The system has a hierarchical organizational structure:
- **Polos** (Oil & Gas Hubs) → have many Campos, Instalações, Equipamentos
- **Campos** (Fields) → belong to Polos
- **Instalações** (Installations) → belong to Polos and optionally Campos
- **Equipamentos** (Equipment) → belong to Instalações and Polos
- **Pontos de Medição** (Measurement Points) → belong to Instalações and Polos, reference multiple equipment via serial numbers

Equipment can have:
- **Plano de Calibração** (Calibration Plans)
- **Histórico de Calibrações** (Calibration History)
- **Calendário de Calibrações** (Calibration Calendar/Schedule)
- **Execução de Calibrações** (Calibration Execution) - stores last 3 certificates with technical data
- **Certificados de Calibração** (Calibration Certificates) - historical records with ordem_certificado
- **Placas de Orifício** (Orifice Plates)
- **Trechos Retos** (Straight Sections)
- **Medidores Primários** (Primary Meters)

Other entities:
- **Poços** (Wells) → have **Testes** (Well Tests)
- **Válvulas** (Valves) → with leak testing
- **Controle de Incertezas** (Uncertainty Control) → for measurement points
- **Análises Químicas** (Chemical Analysis) - multiple types: generic, chromatography, PVT
- **Gestão de Cilindros** (Cylinder Management)
- **Sistema de Notificações** (Notification System) - auto-generated alerts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mauricioamorim3r/EquipamentoMedicao](https://github.com/mauricioamorim3r/EquipamentoMedicao) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
