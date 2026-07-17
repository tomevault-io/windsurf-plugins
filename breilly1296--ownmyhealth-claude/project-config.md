---
trigger: always_on
description: Privacy-first HIPAA-compliant health biomarker tracking platform with insurance document management, AI-powered guidance, provider-patient collaboration, and expense tracking. Focused on secure tracking of health metrics with Codex AI educational insights, insurance cost analysis, and provider data sharing via consent-based access control.
---

# OwnMyHealth - Project Context

## What This Is
Privacy-first HIPAA-compliant health biomarker tracking platform with insurance document management, AI-powered guidance, provider-patient collaboration, and expense tracking. Focused on secure tracking of health metrics with Codex AI educational insights, insurance cost analysis, and provider data sharing via consent-based access control.

## Tech Stack
- **Frontend**: React 18 + Vite 7.3 + TypeScript + Tailwind CSS
- **Backend**: Node.js + Express 4.18 + TypeScript
- **Database**: PostgreSQL (Cloud SQL) + Prisma ORM
- **Auth**: JWT access tokens + refresh tokens (DB-backed sessions) + CSRF double-submit cookie
- **Encryption**: AES-256-GCM for all PHI (per-user keys via PBKDF2-SHA512)
- **AI**: Anthropic Codex API (biomarker guidance, cost analysis, document extraction)
- **OCR**: Google Document AI (scanned lab reports)
- **Email**: SendGrid (verification, password reset)
- **File Storage**: Google Cloud Storage (lab reports, SBC documents)
- **Testing**: Vitest (frontend and backend)
- **Deployment**: GCP Cloud Run (backend) + GCS bucket (frontend) + Cloud SQL (database)

## Current Features
- **Biomarker Tracking**: Manual entry, history, trends, normal ranges, AI educational guidance
- **DEXA Scan Support**: Upload and track bone density measurements
- **Insurance Management**: SBC document upload (Codex AI extraction), plan comparison, benefit search
- **Expense Tracking**: Projections, actuals, AI-powered cost analysis
- **Health Goals**: Goal tracking with progress notes and history
- **Health Needs**: Track health needs with status, type, and urgency
- **Provider Collaboration**: Consent-based provider-patient data sharing with granular permissions
- **File Management**: Lab report upload (PDF parsing + OCR), GCS storage, signed URL downloads
- **Admin Panel**: User management, audit log viewer, system health stats
- **Demo Mode**: Demo account for development/testing (blocked in production)
- **Audit Logging**: HIPAA-compliant access logging with 7-year retention
- **Email Notifications**: Email verification, password reset via SendGrid

## Removed Features (Jan 2025)
- ~~Health Scoring~~ - 0-100 health scores, risk assessments (dashboard shows "Biomarkers in Range %" instead — a simple in-range ratio, not the removed scoring system)
- ~~CMS Marketplace Integration~~ - healthcare.gov plan search
- ~~Provider Directory~~ - doctor search and recommendations

## Project Structure
```
src/
├── components/
│   ├── analytics/      # Trend charts (TrendChart, BiomarkerChart)
│   ├── auth/           # Login, registration, email verification, password reset
│   ├── biomarkers/     # Biomarker display, entry, modals, AI guidance
│   ├── common/         # Shared UI (Button, Modal, RoleGuard, etc.)
│   ├── dashboard/      # Main dashboard
│   ├── files/          # File management (list, download, delete)
│   ├── insurance/      # Insurance hub, plan management, SBC upload
│   ├── settings/       # Data export, account deletion
│   ├── trends/         # Trend visualizations
│   └── upload/         # File upload components
├── contexts/           # React contexts (Auth, Theme)
├── services/
│   └── api/            # API client modules (13 files)
│       ├── client.ts   # Base HTTP client (axios + interceptors)
│       ├── auth.ts     # Auth endpoints
│       ├── biomarkers.ts
│       ├── insurance.ts
│       ├── expenses.ts
│       ├── healthGoals.ts
│       ├── healthNeeds.ts
│       ├── files.ts
│       ├── upload.ts
│       ├── provider.ts
│       ├── patient.ts
│       ├── admin.ts
│       ├── settings.ts
│       └── index.ts    # Re-exports all modules
├── types/              # TypeScript interfaces
└── data/               # Sample data, nav config

backend/src/
├── controllers/        # Route handlers (10 files)
│   ├── authController.ts
│   ├── biomarkerController.ts
│   ├── expenseController.ts
│   ├── fileController.ts
│   ├── healthGoalsController.ts
│   ├── healthNeedsController.ts
│   ├── insuranceController.ts
│   ├── settingsController.ts
│   └── uploadController.ts
├── middleware/          # Security middleware (8 files)
│   ├── auth.ts         # JWT verification
│   ├── csrf.ts         # CSRF double-submit cookie
│   ├── rateLimiter.ts  # 8 named rate limiters
│   ├── rbac.ts         # Role-based access control
│   ├── demoProtection.ts # Demo account restrictions
│   └── ...
├── routes/             # API route definitions (13 files, 60+ endpoints)
├── services/           # Business logic (18 files)
│   ├── encryption.ts   # PHI encryption (AES-256-GCM)
│   ├── userEncryption.ts # Per-user keys (PBKDF2-SHA512)
│   ├── auditLog.ts     # HIPAA audit trail + cleanup scheduler
│   ├── authService.ts  # Auth logic + session cleanup scheduler
│   ├── database.ts     # Prisma client + RLS context
│   ├── claudeExtraction.ts # Codex AI document extraction
│   ├── sbcExtraction.ts # SBC-specific Codex extraction
│   ├── storageService.ts # Google Cloud Storage

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [breilly1296/OwnMyHealth-Claude](https://github.com/breilly1296/OwnMyHealth-Claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
