---
trigger: always_on
description: <!-- BEGIN:nextjs-agent-rules -->
---

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->
# AGENTS.md — Star Tech India CRM

## Final Technical Product & Implementation Specification

> **Status:** Final implementation contract
> **Product:** Star Tech India CRM
> **Application type:** Internal responsive web application
> **Primary users:** Owner, Admin, Sales, Project, Delivery and Accounts teams
> **Deployment:** Vercel + Supabase
> **Mobile application:** Not in current scope
> **UI library:** shadcn/ui only
> **Chart library:** Apache ECharts only

---

# 1. Purpose of This File

This file is the implementation source of truth for any coding agent working on the Star Tech India CRM.

The implementing agent must follow this document before introducing:

* new packages
* new UI patterns
* new database tables
* new status values
* new roles
* new permissions
* new integrations
* new infrastructure
* new business workflows

Do not invent architecture because another approach is personally preferred.

If requirements are ambiguous, preserve the architecture and business rules defined here rather than introducing a conflicting pattern.

---

# 2. Product Objective

Star Tech India CRM manages the complete business lifecycle:

```text
MARKETING
    ↓
LEAD
    ↓
SALES
    ↓
QUALIFICATION
    ↓
MEETING / DEMO
    ↓
REQUIREMENT
    ↓
ESTIMATE / QUOTATION
    ↓
NEGOTIATION
    ↓
WON / LOST
    ↓
CLIENT
    ↓
PROJECT
    ↓
TEAM EXECUTION
    ↓
DELIVERY
    ↓
INVOICE
    ↓
PAYMENT
    ↓
SUPPORT / CLOSURE
```

The CRM must connect these stages instead of creating unrelated modules.

The Owner must be able to understand the complete story of a customer from initial marketing source through payment and project completion.

---

# 3. Non-Negotiable Technology Decisions

## Application

* Next.js App Router
* React
* TypeScript with strict mode
* pnpm package manager

## Hosting

* Vercel

## Backend

* Supabase PostgreSQL
* Supabase Auth
* Supabase Row Level Security
* Supabase Storage
* Supabase Edge Functions
* PostgreSQL functions/RPC
* Supabase Cron for scheduled work
* Supabase Realtime only where justified

## UI

**shadcn/ui only.**

Use official shadcn components and current official shadcn dashboard/sidebar block patterns wherever appropriate.

Do not introduce another general-purpose component library.

## Charts

**Apache ECharts only.**

Do not use:

* Recharts
* Chart.js
* ApexCharts
* Highcharts
* Nivo
* Victory
* Plotly
* Tremor charts
* shadcn Chart/Recharts implementation

Important:

shadcn's normal Chart abstraction may use Recharts. **Do not use it.**

Charts should instead use the `echarts` package directly inside a small reusable internal React component.

Example internal component:

```text
components/charts/e-chart.tsx
```

The surrounding cards, dropdowns, filters and layout must still use shadcn/ui.

## Server State

* TanStack Query

## Tables

* TanStack Table
* shadcn Table

## Forms

* React Hook Form
* Zod
* shadcn Form

## Small Client UI State

* Zustand only when genuinely useful

Do not use Zustand as a replacement for server state.

## Icons

* Lucide React only

## Email

* Brevo Transactional Email API

## Integrations

* Star Tech India website lead forms
* Meta/Facebook Lead Ads
* Instagram Lead Ads where supported through Meta

## Redis

**Do not add Redis for MVP.**

---

# 4. Explicitly Out of Scope

Do not build these in the current version:

* Android application
* iOS application
* desktop native application
* multi-tenant SaaS architecture
* subscription billing
* payroll
* HRMS
* complete ERP
* complete procurement ERP
* complex inventory system
* customer mobile application
* WhatsApp CRM inbox
* AI agents
* AI lead scoring
* Redis infrastructure
* Trigger.dev
* Kafka
* RabbitMQ
* microservices
* Kubernetes
* custom workflow builder

The system is an internal Star Tech India web CRM.

---

# 5. UI Architecture Rules

All non-chart UI must use shadcn components where a suitable component exists.

Preferred components include:

* Sidebar
* Breadcrumb
* Button
* Card
* Badge
* Avatar
* Input
* Textarea
* Checkbox
* Radio Group
* Switch
* Select
* Command
* Popover
* Calendar
* Date Picker patterns
* Form
* Table
* Tabs
* Dialog
* AlertDialog
* Sheet
* Drawer
* DropdownMenu
* ContextMenu
* Tooltip
* HoverCard
* Separator
* Skeleton
* ScrollArea
* Collapsible
* Accordion
* Progress
* Pagination
* Sonner
* Empty state patterns
* Command palette patterns

Do not manually recreate a component already available through shadcn.

---

# 6. Dashboard Shell

Start from the current official shadcn dashboard/sidebar block architecture rather than hand-building a completely different shell.

The application should use:

```text

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dhanushkumar-merndev/startechindiacrm](https://github.com/dhanushkumar-merndev/startechindiacrm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
