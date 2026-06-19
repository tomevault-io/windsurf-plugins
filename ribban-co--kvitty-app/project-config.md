---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Kvitty is a comprehensive Swedish bookkeeping and invoicing SaaS application. It supports both simple receipt tracking ("simple" mode) and full double-entry bookkeeping ("full_bookkeeping" mode) with advanced features including:

- **Invoicing**: Full invoice lifecycle with PDF generation, email delivery, Swish QR codes, and public invoice links
- **Payroll**: Swedish payroll with tax calculations, AGI XML generation for Skatteverket, and salary statements
- **Bank Integration**: Bank transaction import with automated matching and duplicate detection
- **Tax Compliance**: ROT/RUT deductions, margin scheme for used goods, reverse charge, and Peppol e-invoicing
- **Reporting**: Income statement, balance sheet, VAT reports with BAS account grouping
- **Annual Closing**: Multi-step bokslut wizard with K1/K2/K3 packages
- **Email Inbox**: Automated document processing from workspace-specific email addresses
- **AI Features**: Bank transaction extraction, bookkeeping assistant, and receipt image analysis

## Important Rules

**Never run `pnpm dev` or `pnpm build` automatically.** These commands should only be run manually by the user.

**Never create `index.ts` barrel export files.** Import directly from the specific file instead (e.g., `import { hasFeature } from "@/lib/feature-flags/utils"` not `from "@/lib/feature-flags"`).

## Development Commands

**IMPORTANT:** Never run `pnpm dev` or `pnpm build` automatically. These commands should only be run manually by the user.

```bash
pnpm dev              # Start development server
pnpm build            # Production build
pnpm lint             # Run ESLint
pnpm type-check       # TypeScript type checking

# Database (Drizzle + PostgreSQL)
pnpm db:push          # Push schema changes to database
pnpm db:generate      # Generate migrations
pnpm db:migrate       # Run migrations
pnpm db:studio        # Open Drizzle Studio
pnpm db:wipe          # Wipe database (scripts/wipe-db.ts)

# Demo & Setup
pnpm demo:populate    # Populate workspace with demo data
pnpm fetch-tax-tables # Fetch Swedish tax tables from Skatteverket
```

## Architecture

### Tech Stack
- **Framework**: Next.js 16.1.1 with App Router + React 19.2.3
- **Database**: PostgreSQL with Drizzle ORM 0.45.1
- **API**: tRPC 11.8 with React Query + SuperJSON transformer
- **Authentication**: better-auth 1.4 (magic link, email OTP, Google OAuth)
- **Styling**: Tailwind CSS v4 + shadcn/ui components
- **File Storage**: Abstracted provider system (Local filesystem or AWS S3 + CloudFront)
- **AI**: Groq SDK with AI SDK (3 models: GPT-OSS-120B, Kimi-K2, Llama-4-Maverick)
- **Forms**: react-hook-form 7.69 + Zod 4.3 validation
- **Tables**: @tanstack/react-table 8.21 with server-side pagination
- **Dates**: date-fns 4.1
- **PDF**: jsPDF 3.0 for invoice/salary statement generation
- **Charts**: recharts 3.6
- **Animations**: Motion 12.23
- **Icons**: @phosphor-icons/react 2.1 + lucide-react
- **File Uploads**: react-dropzone 14.3
- **Drag & Drop**: @dnd-kit 6.3
- **Flow Diagrams**: @xyflow/react 12.10
- **Theming**: next-themes 0.4
- **QR Codes**: qrcode 1.5
- **Email**: UseSend via Nodemailer 7.0
- **XML Parsing**: fast-xml-parser 5.3 for SIE5/AGI/Peppol
- **Encoding**: iconv-lite for SIE4 import (CP437, Latin-1, UTF-8)
- **Security**: Encryption for sensitive data (personal numbers), botid for bot detection

### Route Structure
```
app/
├── (auth)/              # Public authentication pages
│   ├── login/           # Email/password + magic link + Google OAuth
│   ├── signup/          # New user registration
│   ├── otp/             # Email OTP verification
│   └── magic-link-sent/ # Magic link confirmation
├── (dash)/              # Protected dashboard (requires auth)
│   ├── dashboard/       # User dashboard (workspace selector)
│   ├── user/
│   │   └── installningar/ # User settings
│   └── [workspaceSlug]/   # Workspace-scoped pages
│       ├── page.tsx         # Workspace dashboard (metrics, activity)
│       ├── bank/            # Bank accounts & transaction import
│       ├── fakturor/        # Invoices (draft, sent, paid)
│       │   └── [invoiceId]/ # Invoice detail & edit
│       ├── kunder/          # Customer management
│       ├── produkter/       # Product catalog
│       ├── personal/        # Employee management
│       │   └── lon/         # Payroll runs & AGI generation
│       │       └── [runId]/ # Payroll run detail
│       ├── verifikationer/  # Journal entries (full_bookkeeping mode)
│       ├── transaktioner/   # Bank transactions (simple mode)
│       ├── inbox/           # Email inbox for document processing
│       ├── medlemmar/       # Workspace members & invitations
│       ├── perioder/        # Fiscal periods management
│       ├── bokslut/         # Annual closing wizard
│       ├── installningar/   # Workspace settings
│       └── rapporter/       # Financial reports
│           ├── resultat/    # Income statement (Resultaträkning)
│           ├── balans/      # Balance sheet (Balansräkning)
│           └── moms/        # VAT report (Momsrapport)
├── (web)/               # Public marketing pages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ribban-co/kvitty-app](https://github.com/ribban-co/kvitty-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
