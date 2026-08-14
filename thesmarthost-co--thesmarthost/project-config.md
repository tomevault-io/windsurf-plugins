---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **HostMetrics Frontend** - Property management reporting platform built with Next.js, TypeScript, and Tailwind CSS

**Last Updated:** January 22, 2026

---

## Quick Reference

### Development Commands

```bash
# Start development server (with Turbopack)
npm run dev
# → Runs on http://localhost:3000

# Build for production (ALWAYS run before pushing)
npm run build

# Start production server
npm start
```

**Important:** Always run `npm run build` after changes before pushing to verify no TypeScript or build errors.

---

## Project Context

### What is HostMetrics?

Property management reporting platform for short-term rental managers.

- **Client:** TheSmartHost Co. Inc (Luis Torres)
- **Team:** Mark Cena (Calgary) + Hussein Saab (Toronto)
- **Timeline:** Oct 7 - Dec 20, 2025 (10 weeks)
- **Status:** Feature Complete - All core features implemented

**Goal:** Automate monthly financial reports for property owners (reduce from 2-4 hours → 10 minutes per client)

---

## Tech Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| Next.js | 16.1.0 | App Router with Turbopack |
| React | 19.2.3 | UI Framework |
| TypeScript | 5.x | Type Safety (strict mode) |
| Tailwind CSS | 4.x | Styling |
| Zustand | 5.0.8 | State Management |
| Recharts | 3.6.0 | Analytics Charts |
| Framer Motion | 12.x | Animations |
| Supabase SSR | 0.7.0 | Authentication |
| Heroicons | 2.2.0 | Icons |
| React Icons | 5.5.0 | Additional Icons |
| React Markdown | 10.1.0 | Markdown Rendering (AI Insights) |

**Backend:** Express.js + PostgreSQL (separate repository: `thesmarthost-backend`)

---

## Architecture Patterns

### 1. Route Groups for Access Control

```
app/
├── (prelogin)/     # Public pages (login, signup, about, contact)
├── (user)/         # Protected pages
│   └── property-manager/
│       ├── analytics/
│       ├── bookings/
│       ├── clients/
│       ├── dashboard/
│       ├── expenses/
│       ├── incoming-bookings/
│       ├── properties/
│       ├── reports/
│       ├── settings/
│       └── upload-bookings/
├── api/auth/       # Auth callback routes
├── forbidden/      # Access denied page
├── error.tsx       # Error boundary
└── not-found.tsx   # 404 page
```

**Layout inheritance:**
- `(prelogin)` → uses PreNavbar
- `(user)` → uses UserNavbar
- `property-manager` → adds ManagerSidebar

### 2. Component Organization by Resource + Action

```
components/
├── [resource]/
│   ├── create/
│   │   └── create[Resource]Modal.tsx
│   ├── update/
│   │   └── update[Resource]Modal.tsx
│   ├── delete/
│   │   └── delete[Resource]Modal.tsx
│   ├── preview/
│   │   └── preview[Resource]Modal.tsx
│   └── import/
│       ├── bulk[Resource]Modal.tsx
│       └── steps/
└── shared/
    ├── modal.tsx
    ├── notification.tsx
    ├── TableActionsDropdown.tsx
    ├── FloatingActionButton.tsx
    └── LogoutModal.tsx
```

**Rules:**
- Folders use lowercase-hyphenated names (`client-agreement/`)
- Files use PascalCase (`createPropertyModal.tsx`)
- All interactive components must have `'use client'` directive

### 3. Service Layer Architecture

```
services/
├── apiClient.ts              # Fetch wrapper with credentials
├── [resource]Service.ts      # API functions (GET, POST, PUT, DELETE)
└── types/
    └── [resource].ts         # TypeScript interfaces
```

### 4. API Response Contract

All backend endpoints follow this format:

```typescript
// Success
{ status: 'success', data: { ... } }

// Error
{ status: 'failed', message: 'Human-readable error' }

// Frontend handling
const res = await createResource(payload)
if (res.status === 'success') {
  showNotification('Success', 'success')
} else {
  showNotification(res.message || 'Failed', 'error')
}
```

### 5. State Management Strategy

**Zustand Stores (3 total):**
- `useUserStore` - User profile and auth state (persisted to localStorage)
- `useNotificationStore` - Toast notifications (not persisted)
- `useAnalyticsStore` - Analytics filters and data (not persisted)

**Local State (`useState`):** Form inputs, modals, fetched data

---

## Complete Component Inventory (98 components)

### Analytics Components (8 files)
```
components/analytics/
├── AnalyticsWidget.tsx           # Main composite component (full & compact modes)
├── DrillDownModal.tsx            # Modal for viewing filtered bookings
└── shared/
    ├── AIInsightsCard.tsx        # AI-generated weekly insights (markdown)
    ├── AnalyticsFilters.tsx      # Date range presets + property/channel filters
    ├── BreakdownTabs.tsx         # Property/Channel tabs with table/chart views
    ├── KPICard.tsx               # Individual metric card with delta
    ├── KPIGrid.tsx               # Grid of 7 KPI cards
    ├── TimelineChart.tsx         # Recharts line/area/bar chart
    └── index.ts                  # Barrel exports
```

### Booking Components (5 files)
```
components/booking/
├── create/createBookingModal.tsx
├── delete/deleteBookingModal.tsx
├── import/ImportHostawayBookingsModal.tsx
├── preview/previewBookingModal.tsx
└── update/updateBookingModal.tsx
```

### Client Components (10 files)
```
components/client/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheSmartHost-Co/thesmarthost](https://github.com/TheSmartHost-Co/thesmarthost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
