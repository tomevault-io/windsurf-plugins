---
trigger: always_on
description: Gymly is a **multi-tenant SaaS gym management platform**. Every gym is an isolated
---

# AGENTS.md — Gymly Security & Safety Rules
# Project: gymly-app-06 · Stack: React 19 + Vite 5 + Firebase 12 + Tailwind 3
# Last updated: 2026-06-04
# READ THIS ENTIRE FILE BEFORE WRITING ANY CODE OR RULE

---

## 1. IDENTITY OF THIS PROJECT

Gymly is a **multi-tenant SaaS gym management platform**. Every gym is an isolated
tenant. A security mistake in one gym's data can expose every other gym's data.
There are currently live gym owners and members on production. No change may break
existing functionality or cause data loss.

Firebase project ID: `gymly-app-06`
Frontend deploy: Vercel (gymly.online)
Functions runtime: Node.js 18, CommonJS (.js), NOT TypeScript

---

## 2. NON-NEGOTIABLE RULES — NEVER VIOLATE THESE

These rules apply to every file, every prompt, every diff. No exceptions.

### 2.1 Firestore data model constraints

```
RULE: Staff and members are created with addDoc() — their Firestore doc IDs are
      auto-generated random strings. They are NOT equal to Firebase Auth UIDs.
      Only gym owner documents use setDoc(doc(db,'users',uid)) where uid matches Auth.

CONSEQUENCE: Never write Firestore rules or queries that assume
             doc.id == request.auth.uid for staff or member documents.
             Use gym_id scoping instead.

RULE: Membership plans are embedded arrays in gym.settings.plans.
      Do NOT create a separate /plans collection.

RULE: profile_photo must always be a Firebase Storage download URL string.
      Never store base64 image data in Firestore documents.
      Firestore document size limit is 1MB — base64 photos exceed this.

RULE: Never store undefined values in Firestore. Use null.

RULE: gym_id must be validated server-side on all sensitive collection writes.
      Client-supplied gymId cannot be trusted without a server-side ownership check.
```

### 2.2 Authentication and mock mode

```
RULE: The mockRole localStorage bypass MUST ONLY exist in development builds.
      Every mockRole check in any file must be wrapped in:
        if (import.meta.env.DEV) { ... }
      Vite tree-shakes this in production. Verify by checking that import.meta.env.DEV
      is the outermost condition before any localStorage.getItem('mockRole') call.

RULE: Never add new mockRole checks to any file without the DEV guard.

RULE: onAuthStateChanged must NEVER be inside a DEV guard.
      It must remain unconditional. If it is guarded, all production users are locked out.

AFFECTED FILES:
  src/context/AuthContext.jsx         — mockRole block wrapped in DEV guard ✓ DONE
  src/firebase/firestore.js           — isMock() prefixed with DEV guard ✓ DONE
  src/firebase/firestore-payments.js  — isMock() prefixed with DEV guard ✓ DONE
```

### 2.3 Firestore security rules

```
RULE: Deny by default. Every collection must have an explicit rule.
      Never rely on implicit denial — always write allow rules explicitly.

RULE: The /users collection read rule must ALWAYS require:
        request.auth != null
      AND either:
        request.auth.uid == uid  (own document)
      OR:
        caller's gym_id == resource's gym_id  (same-gym staff access)
      Public unauthenticated reads of /users are PERMANENTLY FORBIDDEN.

RULE: The /users collection update rule must ALWAYS block:
        'role' in request.resource.data
        'gym_id' in request.resource.data
      Role and gym_id are server-managed fields. No client write may set them.

RULE: The /users collection delete rule must ALWAYS be:
        allow delete: if false;
      Member deletion goes through the softDeleteMember Cloud Function only.

RULE: /gyms writes must require request.auth.uid == resource.data.owner_id.
      No cross-gym gym document writes are ever permitted.

RULE: /payments, /invoices, /attendance_logs, /whatsapp_logs, /invoice_counter,
      /message_retry_queue, /message_logs, /numbering_settings, /serial_counters
      must ALL have gym_id ownership checks. Use:
        get(/databases/$(database)/documents/users/$(request.auth.uid)).data.gym_id
          == resource.data.gym_id

RULE: /used_coupons allow write: if false — all writes go through Cloud Function.

RULE: /coupons allow read, write: if false — server-only via Admin SDK.

RULE: /deleted_members/{gymId}/bin/{memberId}
        allow read: owner and manager in same gym only
        allow write: if false — Cloud Function only

RULE: /audit_logs/{gymId}/events/{eventId}
        allow read: owner and manager in same gym only
        allow write: if false — append-only via Cloud Function, NEVER from client

RULE: /leads allow create with strict field validation only:
        keys must be subset of [name, phone, email, gym_id, created_at, message]
        name and phone must be strings under 100 characters

RULE: kiosk_devices allow update only if device_secret in request matches stored value.
      attendance_sessions writes must validate device_secret against paired kiosk doc.
      NEVER allow update: if true on kiosk_devices.
      NEVER allow read, write: if true on attendance_sessions.

RULE: After every firestore.rules change, run:
        npx firebase-tools rules:check --project gymly-app-06
      NEVER deploy rules that fail the check.
```

### 2.4 Cloud Functions

```
RULE: All Cloud Functions are CommonJS (.js) files.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gravitycomedia-droid/gymly-app](https://github.com/gravitycomedia-droid/gymly-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
