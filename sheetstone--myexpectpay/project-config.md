---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

**MyExpertPay** is a full-stack payment management portal for Expertpay account holders. The service allows employers to pay employees via debit cards; this portal lets users view balances, manage bank accounts, manage child-support cases and recipients, and review payment history.

This is a greenfield rewrite of an existing React + Firebase app into a **Next.js + Firestore** monolith deployed on **Firebase App Hosting**. The goal is a production-quality application — prioritise correctness, security, and type safety.

## Tech Stack

### Framework
- **Next.js 14+** — App Router, TypeScript strict mode
- **React 18** — Server Components (default) + Client Components (`"use client"`) where interactivity is needed

### Frontend (within Next.js)
- **Routing:** Next.js App Router file-based routing
- **Server state:** TanStack Query (client components) + React Server Components (server reads)
- **UI state:** React Context or `useState` — no Zustand (App Router reduces global state needs)
- **Forms:** React Hook Form + **Zod** resolver (unified validation library)
- **UI components:** Shadcn/UI + custom CSS Modules
- **Styling:** Tailwind CSS + CSS Modules (`.module.css` per component)
- **Charts:** Recharts
- **i18n:** react-intl (EN, DE, ES)
- **Testing:** Vitest + React Testing Library

### Backend (Next.js Route Handlers + Server Actions)
- **API:** Next.js Route Handlers (`src/app/api/...`) — replaces Express
- **Database:** Firestore via **Firebase Admin SDK** (server-side only — no client SDK Firestore access)
- **Auth verification:** Firebase Admin SDK — verifies session cookie on every server request
- **Validation:** Zod (all Route Handler request bodies and params)
- **Testing:** Vitest + Firebase Emulator Suite (Firestore + Auth emulators)

### Infrastructure
- **Auth:** Firebase Authentication (Google OAuth + email/password)
- **Session management:** Firebase session cookies (server-side; `firebase-admin` creates/verifies via `auth().createSessionCookie`)
- **Database:** Firestore (Google-managed, encrypted at rest)
- **Hosting:** Firebase App Hosting (Next.js SSR/SSG/ISR via Cloud Run)
- **CI/CD:** GitHub Actions → Firebase App Hosting GitHub integration
- **Secrets:** Firebase App Hosting environment secrets (no `.env` in repo)

## Architecture

### Core User Flows

1. **Login** → Google OAuth or email/password → dashboard
2. **Dashboard** → account summary + payment chart + activity calendar + recent messages
3. **Bank Accounts** → list → add (routing + account number + type) → edit / verify / delete
4. **Cases** → list → add (case number + NCP + children[]) → edit / delete
5. **Recipients** → list → add (name + email + case) → edit / delete
6. **Payments** → list + filter (date range + status) → send money / request money
7. **Messages** → full inbox, read/unread state
8. **Profile** → edit display name, change password
9. **Settings** → language switcher, delete account

### Request Flow

```
[Browser]
  │
  ├─ Server Components (RSC)  →  Firebase Admin SDK  →  Firestore
  │
  └─ Client Components
       └─ fetch() / TanStack Query  →  Route Handlers (`/api/...`)
                                          └─ verify session cookie
                                          └─ Firebase Admin SDK  →  Firestore
```

### Auth Flow

```
[User] → [Firebase Auth client SDK (Google popup / email+pw)] → [ID Token]
       → [POST /api/auth/session]  →  Firebase Admin: verifyIdToken + createSessionCookie
       → [session cookie set on browser]
       → [middleware.ts verifies cookie on every protected route]
```

### Firestore Data Model

All user data lives in subcollections under `users/{uid}/` — this makes user scoping implicit and simple.

```
users/{uid}
  ├── bankAccounts/{bankId}
  │     fields: bankName, nickname, routingNumber (encrypted), accountNumber (encrypted),
  │             accountNumberLast4, accountType, verified, isPrimary,
  │             receivePayments, sendPayments, createdAt, updatedAt
  │
  ├── cases/{caseId}
  │     fields: caseNumber, ncpName, children (string[]), createdAt, updatedAt
  │
  ├── recipients/{recipientId}
  │     fields: firstName, lastName, email, caseId, createdAt, updatedAt
  │
  ├── payments/{paymentId}
  │     fields: amount, bankId, caseNumber, recipientId, recipientName,
  │             paymentDate, status, type, note, createdAt
  │
  └── messages/{messageId}
        fields: sender, subject, body, isRead, createdAt
```

### Project Structure

```
myexpectpay/
├── src/
│   ├── app/
│   │   ├── (auth)/                  ← unauthenticated layout group
│   │   │   ├── layout.tsx
│   │   │   ├── login/page.tsx
│   │   │   ├── register/page.tsx
│   │   │   └── forgot-password/page.tsx
│   │   ├── (dashboard)/             ← authenticated layout group
│   │   │   ├── layout.tsx           ← auth guard + shell nav
│   │   │   ├── page.tsx             ← dashboard
│   │   │   ├── bank-accounts/
│   │   │   ├── cases/
│   │   │   ├── recipients/
│   │   │   ├── payments/
│   │   │   ├── messages/
│   │   │   ├── profile/
│   │   │   └── settings/
│   │   ├── api/
│   │   │   ├── auth/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sheetstone/myexpectpay](https://github.com/sheetstone/myexpectpay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
