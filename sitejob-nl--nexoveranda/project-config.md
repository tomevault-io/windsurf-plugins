---
trigger: always_on
description: > Reference document for Codex sessions on this repo. Read this first before exploring or making changes.
---

# AGENTS.md — Nexo Veranda

> Reference document for Codex sessions on this repo. Read this first before exploring or making changes.
> Last updated: 2026-04-17 — backend-audit branch merged to main (16 fixes); §7 Known Issues re-verified.

---

## 1. Project Overview

**Nexo Veranda** is a digital ecosystem for an international veranda / outdoor-room manufacturer (client: Mehmet @ Nexo Veranda, NL). The product is a single React SPA + a Supabase backend, but functionally it is **four integrated modules** separated by URL prefix and per-module layout:

| Module | Prefix | Purpose | Status |
|---|---|---|---|
| **A. Webshop + 3D Configurator** | `/`, `/producten/*`, `/configurator`, `/:slug` | Public marketing site, product catalog, 3D wizard, hybrid checkout (direct buy vs. quote request), regional SEO landing pages, AI chatbot | ~95% |
| **B. Admin "Het Brein"** | `/admin/*` | Quotes, orders, invoices, customers, products, suppliers, planning, messaging hub (WhatsApp + email + tickets), Exact Online sync, audit log | ~92% |
| **C. Personeel (Staff)** | `/personeel/*` | Field service: workorders 2.0 (photos, signature, checklist), planning view, time entries, leave requests, expense claims, payslips, onboarding | ~92% |
| **D. Mijn Nexo (Customer Portal)** | `/mijn-account/*` | Order timeline, quotes, invoices, documents, service tickets, Nexo Academy (videos/guides), AI chatbot, push opt-in, notification center | ~92% |

**Core workflows the system supports today:**

1. **Quote → Order → Production → Install → Invoice.** Customer requests quote (anon insert into `quotes`) → admin processes in `/admin/quotes` → on acceptance converts to order → planner schedules install (`planning_items`) → staff completes work order → admin generates invoice (synced to Exact Online).
2. **Customer self-service.** Customer logs in, watches order timeline update, downloads documents (RLS-filtered), browses Academy, opens service tickets, chats with the AI bot.
3. **Staff field workflow.** Staff opens assigned workorder on phone (PWA), uploads photos, captures signature, completes checklist → completion auto-creates a `time_entries` record.

Tech: React 18 + TS + Vite + Tailwind/shadcn + Supabase. PWA-installable. 5 languages (NL/EN/DE/FR/ES). Built originally in Lovable; now being finished in Codex + VS Code.

---

## 2. Tech Stack & Architecture

### Exact versions (from `package.json`)

- **React** 18.3.1, **React-DOM** 18.3.1, **React Router** 6.30.1
- **TypeScript** 5.8.3
- **Vite** 5.4.19, **vite-plugin-pwa** 1.2.0
- **Tailwind CSS** 3.4.17, **shadcn/ui** (Radix primitives)
- **@supabase/supabase-js** 2.94.0
- **@tanstack/react-query** 5.83.0 (used in 52 files)
- **Zustand** 5.0.11 (only `useStickyBarStore` — UI state only)
- **react-hook-form** 7.61.1, **@hookform/resolvers** 3.10.0, **zod** 3.25.76
- **i18next** 25.8.4, **react-i18next** 16.5.4, `i18next-browser-languagedetector`
- **three** 0.170.0, **@react-three/fiber** 8.18.0, **@react-three/drei** 9.117.3
- **jspdf** 4.1.0 (invoice + quote PDFs)
- **framer-motion** 12.31.0, **recharts** 2.15.4, **sonner** 1.7.4, **@dnd-kit/\***
- Package manager: **bun** (`bun.lockb` present) — npm also works

### Folder map (`src/`)

```
src/
├── App.tsx                    # Route map (lines 101-197) — single source of truth for URLs
├── main.tsx                   # Entry; mounts QueryClientProvider, Router, i18n
├── pages/
│   ├── admin/                 # Module B — 37 pages (AdminDashboard, AdminQuotes…)
│   ├── staff/                 # Module C — 13 pages (StaffDashboard, StaffWorkOrders…)
│   ├── customer/              # Module D — 15 pages (CustomerDashboard, CustomerAcademy…)
│   ├── Index.tsx              # Module A — homepage
│   ├── ConfiguratorPage.tsx   # Module A — 3D wizard entry
│   ├── ProductsPage.tsx, ProductDetailPage.tsx, CityLandingPage.tsx, …
│   └── (public: Contact, OverOns, Showrooms, Privacy, …)
├── components/
│   ├── admin/                 # 20 admin-specific components (MarginDashboardWidget, OrderTimeline, …)
│   ├── staff/                 # CompleteWorkOrderDialog, …
│   ├── customer/              # Customer portal pieces
│   ├── configurator/          # Module A — Three.js scene + wizard steps
│   │   ├── Scene3DEnhanced.tsx, VerandaModelEnhanced.tsx (R3F; @ts-nocheck)
│   │   ├── scene/{materials,HouseWall,RoofPanels,GlassWalls,Gutter,LamellaRoof}.tsx
│   │   ├── wizard-steps/{StepDimensions,StepFrameColor,StepRoofType,StepGlassWalls,
│   │   │                  StepLighting,StepSunscreen,StepExtras,StepInstallation}.tsx
│   │   ├── ConfiguratorWizard.tsx, PriceSummaryEnhanced.tsx, PDFExport.tsx, CameraControls.tsx
│   │   └── Configurator.tsx   # ⚠ static demo on homepage — NOT wired
│   ├── chatbot/ChatWidget.tsx # ⚠ hardcoded Supabase URL/key (see §7)
│   ├── portfolio/, ui/        # 50+ shadcn components
│   └── (Header, Footer, Hero, Products, Testimonials, ShowroomSection, FAQ, …)
├── hooks/                     # 41 hooks; data-access layer (one per domain)
│   ├── useAuth.ts, useAdminAuth.ts, useStaffAuth.ts, useCustomerAuth.ts
│   ├── useOrders.ts, useQuotes.ts, useInvoices.ts, useCustomers.ts, useSuppliers.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sitejob-nl/nexoveranda](https://github.com/sitejob-nl/nexoveranda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
