---
trigger: always_on
description: Pre-built index files are in `.ai-codex/`. Read these FIRST before exploring the codebase:
---

## Codebase Index
Pre-built index files are in `.ai-codex/`. Read these FIRST before exploring the codebase:

- `.ai-codex/lib.md` -- library exports
- `.ai-codex/schema.md` -- database schema
- `.ai-codex/components.md` -- component tree

# Trakovo — Claude Context

Fleet management platform for vehicle hire bookings, drivers, vendors, and dispatch.
Current version: **v1.15.3**

---

## Stack

- **Framework:** Next.js 14 (App Router, `revalidate = 0` on data pages)
- **Database:** MySQL via raw `mysql2` — no Prisma ORM (removed in v1.3.0; Prisma's Rust binary panics on cPanel shared hosting)
- **Auth:** Custom JWT cookies — separate secrets for admin, vendor, driver (Web Crypto API — works in both Node.js and Edge runtimes)
- **Styling:** Tailwind CSS
- **Language:** TypeScript
- **Hosting:** cPanel shared hosting (CloudLinux + Phusion Passenger)
- **Email:** Microsoft 365 Graph API (primary) with SMTP/Nodemailer fallback
- **Push:** Web Push (VAPID) via `web-push`
- **Key deps:** `mysql2`, `adm-zip`, `nodemailer`, `web-push`, `qrcode`, `date-fns`, `axios`

---

## Project structure

```
src/
  app/
    api/
      admin/          — admin API routes (auth-gated)
        bookings/     — booking CRUD, notes, status, send-quote, driver assign
        drivers/      — driver CRUD, password reset, messages
        users/        — additional admin user CRUD (master-only)
        vehicles/     — vehicle CRUD, next-id helper
        vendors/      — vendor CRUD, password reset, vehicle assignment
        settings/     — general settings, logo, email connections (MS/GC), push, QR, email preview/test
        update/       — OTA update: check, pull, upload, rollback
        login/logout/
        push/         — subscribe, test, vapid-key
        qr/           — QR code for booking app
    admin/            — admin portal pages
      bookings/       — list + detail (notes, status, driver assign, pricing, send-quote)
      calendar/       — calendar view of all bookings
      drivers/        — driver list, new, detail/edit
      users/          — additional admin user management
      vehicles/       — vehicle list, new, edit (media upload, day rates, POA pricing)
      vendors/        — vendor list, new, detail (tabs: info, bookings, vehicles, clients)
      settings/       — general, connections, templates, updates, booking-app (QR)
    api/
      vendor/         — vendor portal API routes
      driver/         — driver portal API routes
      booking/        — public booking submission + ID upload
      vehicles/       — public vehicle availability check
      logo/           — public logo endpoint
      uploads/        — serves uploaded files from UPLOAD_DIR
      maintenance-auth/ — bypass cookie for maintenance/dev mode
    vendor/           — vendor portal pages (dashboard, bookings, clients, calendar, support)
    driver/           — driver portal pages (dashboard, bookings, calendar, messages)
    book/             — public booking flow (vehicle list, per-vehicle booking form)
    vehicles/         — public fleet listing + detail pages
    confirmation/     — post-booking confirmation page
    maintenance/      — maintenance/dev mode lock page
    page.tsx          — homepage (redirects to /book or landing)
  components/
    admin/
      VehicleForm.tsx — shared vehicle create/edit form (media, day rates, POA)
    booking/
      BookingPanel.tsx       — full booking UI state machine
      Calendar.tsx           — date picker calendar
      ConfirmationUploadCard.tsx
      HireAgreementModal.tsx
    ui/
      CalendarView.tsx  — shared calendar grid used in admin/vendor/driver portals
      VehicleCard.tsx
      Nav.tsx / NavWrapper.tsx / Footer.tsx
  lib/
    db.ts             — mysql2 pool + query/queryOne/execute/newId/generatePublicId
    auth.ts           — admin JWT (create, verify, cookie helpers)
    vendor-auth.ts    — vendor JWT
    driver-auth.ts    — driver JWT (8-hour sessions, cookie: apex_driver_session)
    email.ts          — MS Graph (primary) + SMTP fallback, sendBookingNotification, sendCustomerQuote, sendTestEmail
    email-templates.ts     — template rendering with variable substitution + conditionals
    email-template-defaults.ts — default HTML for booking_notification and customer_quote templates
    calendar.ts       — Google Calendar OAuth helpers
    next-update.ts    — OTA update logic (adm-zip extract, backup, swap, Passenger restart)
    site.ts           — getSiteName, getAdminName, getDriverName, getLogoUrl (DB-backed with env fallback)
    push.ts           — web push helpers
    password.ts       — hashPassword / verifyPassword (bcrypt-equivalent)
    uploads.ts        — upload path helpers (uses UPLOAD_DIR env var)
    utils.ts          — formatCurrency, etc.
    api.ts            — shared fetch wrappers for client-side portal API calls
  middleware.ts       — JWT auth guard for /admin, /vendor, /driver; maintenance/dev mode redirect
  types/index.ts      — shared types (Vehicle, Booking, DayRate, HireType, TripLeg, BookingFormState, etc.)
prisma/
  init.sql            — full schema SQL for fresh installs (NOTE: out of sync — see below)
  schema.prisma       — kept for reference only, not used at runtime
```

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SpaceyPuppy/trakovo](https://github.com/SpaceyPuppy/trakovo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
