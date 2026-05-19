---
trigger: always_on
description: This file provides all context needed for Claude Code to continue building Chkin.
---

# CLAUDE.md — Context for AI Coding Assistants

This file provides all context needed for Claude Code to continue building Chkin.

## Session Continuity (READ FIRST)

**Before starting any work, check for active session state:**

1. **Check `.handoff.md`** in project root (if exists)
   - If `Last Updated` < 4 hours ago → Offer to continue previous session
   - If stale → Summarise what was in progress, ask if still relevant

2. **Check recent git history**
   ```bash
   git log --oneline -10
   git status
   ```

3. **When ending a session**, update `.handoff.md` with:
   - What was accomplished
   - What's in progress
   - Files modified
   - Next steps

4. **Check `GAP-ANALYSIS.md`** for tracked improvements
   - If you completed any items listed there, update the document
   - Mark completed items with ✅ and date
   - Update the "Overall Score" if significant progress was made
   - Check GitHub Issues for tracked gaps: `gh issue list --label "gap"`

---

# 1. Project Overview
Chkin is a healthcare digital onboarding system. The main function is to allow patients to submit personal details, medical-aid information, consent, and next-of-kin information via a mobile-friendly digital form, accessed primarily through a QR code in the waiting room.

A practice-facing dashboard allows staff to view submitted forms.

---

# 2. Current State

## 2.1 What's Built
- **Next.js 16** app with TypeScript, Tailwind CSS, ESLint
- **Three-portal route structure**: `/patient`, `/provider`, `/admin`
- **SQLite + Prisma** with Litestream replication to GCS
- **Better Auth** with multi-tenancy (organizations), roles, email verification
- **Docker** configuration for Cloud Run deployment
- **Admin console**: field library, provider/user management, audit logs
- **Provider portal**: form builder, QR codes, submissions management
- **Patient portal**: public form submission, data vault, consent management
- **Mobile app** (Expo/React Native): auth, biometric login, data vault
- **Production deployed** on Google Cloud Run (africa-south1)
- GitHub repo: https://github.com/agroene/chkin

## 2.2 Project Structure
```
app/
├── src/app/
│   ├── (admin)/admin/page.tsx      # Admin Console placeholder
│   ├── (patient)/patient/page.tsx  # Patient Portal placeholder
│   ├── (provider)/provider/page.tsx # Provider Portal placeholder
│   ├── layout.tsx                   # Root layout
│   ├── page.tsx                     # Landing page
│   └── globals.css
├── prisma/schema.prisma             # Empty, ready for models
├── docker-compose.yml               # PostgreSQL service
├── Dockerfile                       # Production build
└── .env.example                     # Environment template
```

## 2.3 Tech Stack (Decided)
| Layer      | Choice                                          |
|------------|-------------------------------------------------|
| Runtime    | Node.js + TypeScript                            |
| Framework  | Next.js 16 (App Router)                         |
| Database   | SQLite + Litestream (replicated to GCS)         |
| ORM        | Prisma (with better-sqlite3 adapter)            |
| Styling    | Tailwind CSS                                    |
| Auth       | Better Auth (multi-tenant, role-based)          |
| Hosting    | Google Cloud Run (africa-south1)                |
| CDN/Proxy  | Cloudflare (DNS, SSL, Worker reverse proxy)     |
| Mobile     | Expo / React Native                             |

---

# 3. Next Steps

1. **CI/CD Updates** - GitHub Actions for Cloud Run deployment
2. **Sprint 3: QR Scanning & Check-In (Mobile)**
3. Add missing secrets (resend-api-key, docuseal keys)

---

# 4. Key Design Decisions

## 4.1 Decided
- Rebuild from scratch (no access to original MVP code)
- Replicate functionality of MVP at **www.chkin.co.za**
- POPIA-compliant handling is non-negotiable
- Dynamic field library (admin can define new fields for providers)
- Database schema grows incrementally with features

## 4.2 Architecture Principles
- Monorepo: Single Next.js app with route groups for portals
- Dynamic fields via JSONB in PostgreSQL
- Static core tables for users, providers, consents, audit logs
- API routes within Next.js (can extract later if needed)

---

# 5. Constraints & Guidelines

## 5.1 Regulatory
- POPIA compliance
- Sensitive personal information (PHI) must be handled securely
- Consent must be explicit and logged
- Access to patient data must be auditable

## 5.2 Operational
- Practices have minimal IT sophistication
- Must work on unstable mobile networks
- QR onboarding must work instantly

### 5.3 Must Do
- Run tests before deploying changes
- Commit with meaningful messages (feat:, fix:, docs:, chore:)
- Update this file when adding significant features

### 5.4 Must Not Do
- Commit `.env` or secrets
- Push directly to production without testing
- Modify database schema without migration plan

### 5.5 Sensitive Data
- `.env` contains API keys — never commit
- Database contains personal transaction data
- Card numbers partially visible in `users` table

---

# 6. Reference Documentation
- `docs/mvp-analysis.md` — Comprehensive analysis of existing MVP
- `.handoff.md` — Original project handoff context

---

# 7. Commands

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agroene/chkin](https://github.com/agroene/chkin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
