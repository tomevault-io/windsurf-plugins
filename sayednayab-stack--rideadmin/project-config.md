---
trigger: always_on
description: You are building **RIDE** (Rezolv Integrated Dispatch Engine), a multi-tenant **B2B Transport Management System** sold to **transport vendors/operators**. The **operator is the tenant and the only UI user**. Their *customers* — airlines, corporates, and the sibling systems **RISMA** (passenger IRROPS), **ROMA** (email parsing), **CLASS** (crew HOTAC) — **never log in**; they interact via **APIs** (mocked here).
---

# CLAUDE.md — RIDE Prototype Context (read on every run)

You are building **RIDE** (Rezolv Integrated Dispatch Engine), a multi-tenant **B2B Transport Management System** sold to **transport vendors/operators**. The **operator is the tenant and the only UI user**. Their *customers* — airlines, corporates, and the sibling systems **RISMA** (passenger IRROPS), **ROMA** (email parsing), **CLASS** (crew HOTAC) — **never log in**; they interact via **APIs** (mocked here).

This is a **clickable prototype for internal validation**, matching the RISMA-demo conventions. **No real backend.** Everything is mocked in-memory with Zustand, seeded with realistic data. The production system (Django/FastAPI + React + Flutter + PostgreSQL + AWS) will reuse the *domain model and flows* defined here, not this throwaway code.

## Tech & conventions
- **Next.js 15 (App Router), React 19, TypeScript (strict), Tailwind CSS, Zustand**. Icons: `lucide-react`. Charts: `recharts`. Maps: **Leaflet or MapLibre with OpenStreetMap tiles only** (never Google).
- **No backend.** Data lives in Zustand stores seeded from `lib/mock/`. You may use Zustand `persist` (localStorage) so refreshes keep state.
- Mock async with small delays where it adds realism ("quoting…", "assigning…", "fetching positions…").
- Build a lightweight in-house UI kit in `components/ui/` (Card, DataTable, Drawer, Modal, Tabs, Badge/StatusBadge, Toast, Button, Input, Select, FormField, and a **PII** mask/tap-to-reveal component).
- Aesthetic: dense **operations / control-room** dashboard. Neutral slate/zinc base, one accent, status colours for state badges. Desktop-first; mobile-tolerant for driver screens.
- Folders: `app/` routes, `components/`, `stores/` (one per domain), `lib/mock/` (seed + fake API), `lib/types/` (shared types).
- **All PII is masked by default and revealed on tap** (name, phone, email, employee id, PNR, licence/registration numbers).

## Multi-tenancy
Top-bar **tenant (operator) switcher** sets `activeTenantId`; all stores scope by it. Seed **2–3 operators** so isolation is visible.

## The two anchoring patterns (do not violate)
1. **Pre-negotiated pricing + quote→book→confirm with price-lock.** Rate cards are pre-negotiated per vendor × customer × vehicle type, effective-dated with version history. An order is **never** created from raw addresses — the caller gets a **priced offer** (a `price_id` + the rate-card `version`), and the order **must cite that `price_id`**. The quoted price is **frozen on the order at booking**. This is why there is no cache/engine divergence and why billing is deterministic.
2. **Pre-flight checks before state changes.** `checkTime` before booking, `checkCancel` before cancelling, `checkUpdate` before editing — each returns whether the action is allowed and its cost/penalty **before** it is attempted.

## Convoy trip model
A `TripRequest` has **one shared ordered stop sequence** and **1..n vehicles** (types may be mixed). **Every vehicle follows the same stops** (a convoy); to diverge, create a new request. Each vehicle has a driver (nullable until assigned) and **0..n pax assigned to that vehicle** (no seat allocation; pax fields may be blank, filled by the calling system).

## Two-level state machine
- **Vehicle status:** `PENDING → ASSIGNED → DRIVER_ACCEPTED|DRIVER_REJECTED → EN_ROUTE_PICKUP → AT_PICKUP → PAX_PICKED(otp) → IN_TRANSIT → AT_DROP → PAX_DROPPED(otp) → COMPLETED`; exceptions `NO_SHOW, BREAKDOWN, ACCIDENT, VEHICLE_SWAP, DELAYED, SOS, CANCELLED`.
- **Trip status (derived):** `DRAFT → CONFIRMED → ASSIGNED → IN_PROGRESS → COMPLETED → BILLED`; plus `CANCELLED`. Roll-up: all vehicles assigned → ASSIGNED; any vehicle moving → IN_PROGRESS; all done → COMPLETED. Exceptions are per-vehicle and surface as **trip alerts** without failing the whole convoy.

## Cancellation (computed, not a policy doc)
Each offer carries a `freeCancellationHours`. **Deadline = pickupTime − freeCancellationHours.** Before the deadline cancellation is free; after it a **configurable penalty** applies. `checkCancel` returns `{ allowed, free, penaltyPct, resultingStatus }`.

## Location typing
Each point has a `locationType`: `AIRPORT | RAIL | HOTEL | CITY | ADDRESS`. Type drives **conditional required fields** (AIRPORT ⇒ `flightNumber` required, optional `terminal`; RAIL ⇒ `trainNumber`; HOTEL detected by a lodging tag) and **pricing eligibility** (exact-address points get a fixed price; imprecise ones show a "from" price).

## Reverse scheduling
For AIRPORT/RAIL **destinations**, compute a **recommended dispatch time** from the flight/train departure time minus travel time and a check-in buffer.

## Authoritative domain model (use across all phases)
```ts
type ID = string;
interface Tenant { id: ID; name: string; legalName: string; baseCity: string; contractCurrency: string; }

// ---- Configuration ----

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sayednayab-stack/rideadmin](https://github.com/sayednayab-stack/rideadmin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
