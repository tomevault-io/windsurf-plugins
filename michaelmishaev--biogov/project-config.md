---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**bioGov** is a planned Progressive Web Application (PWA) designed to guide Israeli small-to-medium business owners through government bureaucracy related to:
- Business registration (VAT dealer status: exempt vs authorized)
- National Insurance (NI) registration for self-employed
- Business licensing (רישוי עסקים)
- Ongoing compliance (VAT deadlines, withholding tax, renewals)
- Document checklists and deep-linking to official government services

**Current Status**: Documentation and planning phase. No source code exists yet—only technical blueprints and requirements.

**Target Users**: Self-employed individuals (עוסק פטור/עוסק מורשה), micro-LTDs (חברה בע״מ קטנה), service trades requiring business licenses.

---

## Technology Stack (Planned)

### Architecture
- **Monorepo**: Turborepo + PNPM workspaces
  - Shared packages for types, utilities, compliance
  - Multiple apps (web, admin, Strapi)
  - Remote caching for faster CI/CD

### Frontend
- **Framework**: Next.js 14+ (App Router) with React 18+, TypeScript
- **PWA**: Service Workers (Workbox) for offline-first functionality
- **Storage**: IndexedDB for offline checklists, compliance calendar, task state
- **State Management**: TanStack Query (React Query) for server state
- **Validation**: Zod for schema validation
- **Localization**: react-i18next (Hebrew RTL-first, English, Russian future)

### Backend & Data
- **Database & Auth**: Supabase (PostgreSQL + Authentication + Storage + Real-time)
  - PostgreSQL for relational data (users, tasks, compliance calendar, audit logs)
  - Built-in authentication (email/password, social login, MFA)
  - Row-Level Security (RLS) for data privacy
  - Real-time subscriptions for live updates
  - File storage for documents
  - Can be self-hosted in Israel for data residency compliance

- **Cache & Jobs**: Redis (Upstash or self-hosted)
  - Job queues for push notification triggers
  - Cache for government API responses (data.gov.il)
  - Session storage for high-performance auth
  - VAT deadline reminder queue

- **CMS**: Strapi (headless CMS)
  - Hebrew micro-guides with government citations
  - Multi-language content management
  - Versioned knowledge cards
  - Non-developer content updates

### External Integrations
- **Government Services**: Deep-linking only (no impersonation). SSO via MyGov authentication
- **Open Data APIs**:
  - ICA company registry (data.gov.il) for form prefill
  - Business licensing dataset for eligibility checks
- **Calendar**: VAT deadline rules (15th/16th/23rd), holiday adjustments, iCal export

---

## Architecture

### Core Components

1. **Eligibility & Pathfinder Engine**
   - Input: Business activity, revenue, location, employees
   - Output: Dealer type (exempt/authorized), licensing requirements, document checklist
   - Data source: Government classification rules

2. **Compliance Calendar Engine**
   - Israeli-specific VAT deadlines (15th/16th/23rd rules)
   - Holiday adjustment table for postponements
   - Push notification triggers
   - iCal export capability

3. **Document Checklist Generator**
   - Maps to business path (e.g., Form 821 for authorized dealer, exempt dealer flow)
   - Dynamic based on user selections
   - Links to official government sources

4. **Company Data Autofill Module**
   - Queries ICA company registry (open data)
   - Prefills business information (non-sensitive data only)

5. **Deep-Link Router**
   - Authenticates via MyGov SSO
   - Routes users to official government services
   - Monitors links for breakage

6. **Knowledge Card System**
   - Educational cards (VAT 18%, e-invoicing rules, licensing requirements)
   - Inline citations to government pages
   - Versioned content with "last updated" timestamps

### Security Model
- **No government service impersonation** (SSO routing to MyGov only)
- **Pseudonymous user IDs** to minimize PII exposure
- **Encrypted storage** of sensitive tokens
- **Access logging** for audit trails
- **Role-based access control** (future multi-tenant)

---

## Database Schema (Proposed)

### Core Tables

**Users**
- `user_id` (PK), `email`, `hashed_password`, `preferred_language`, `accessibility_flags`, `created_at`, `updated_at`, `consent_metadata`

**BusinessProfile**
- `profile_id` (PK), `user_id` (FK), `business_type`, `activity_code`, `municipality_id`, `projected_revenue`, `date_started`, `license_required_flag`

**Tasks**
- `task_id` (PK), `profile_id` (FK), `task_type` (enum: vat_open, ni_register, license_check), `status`, `due_date`, `link_url`, `required_documents` (JSON)

**ComplianceCalendarEntries**
- `entry_id` (PK), `profile_id` (FK), `entry_type` (vat_monthly, withholding_q), `due_date`, `reminder_sent_flag`, `completed_flag`

**DatasetCache**
- `cache_id` (PK), `dataset_name` (ICA_companies, business_licensing_catalog), `version`, `timestamp_fetched`, `data`

**AuditLogs**
- `log_id` (PK), `user_id` (FK), `action_type`, `timestamp`, `metadata` (JSON)

**Preferences**
- `pref_id` (PK), `user_id` (FK), `push_notification_opt_in`, `other_settings`

---

## Regulatory & Compliance Requirements

### Israeli Privacy Law

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MichaelMishaev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
