---
trigger: always_on
description: **MASDIK IQIS Digital Hub** is the official web application for **Masjid Pendidikan Ibnul Qayyim Makassar (MASDIK IQIS)** — an Islamic educational mosque in Makassar, Indonesia. The site serves as the mosque's digital presence and management platform.
---

# CLAUDE.md — MASDIK IQIS Digital Hub

## 1. Project Overview

**MASDIK IQIS Digital Hub** is the official web application for **Masjid Pendidikan Ibnul Qayyim Makassar (MASDIK IQIS)** — an Islamic educational mosque in Makassar, Indonesia. The site serves as the mosque's digital presence and management platform.

**Production URL:** `https://masdik.iqis.sch.id`

### Public-facing features:
- Live prayer times (from aladhan.com API, location: Makassar)
- Mosque activity calendar
- Facility reservation/booking form
- Financial transparency (income/expense ledger + QRIS donation QR)
- DKM organizational structure
- Pustaka (library of documents, videos, audio)
- e-Taklim (online learning — currently "coming soon")

### Admin features (behind auth):
- Manage activities (add/edit/delete)
- Manage reservations (approve/reject)
- Manage financial transactions
- Manage Pustaka items
- QR-code-based attendance system for kajian/rapat/daurah activities

---

## 2. Tech Stack

| Category | Technology |
|---|---|
| Framework | React 18 with TypeScript |
| Build tool | Vite 5 (with `@vitejs/plugin-react-swc`) |
| Routing | React Router DOM v6 |
| Backend/DB | Supabase (PostgreSQL + Auth + RLS) |
| State/data | TanStack React Query v5 |
| UI components | shadcn/ui (Radix UI primitives) |
| Styling | Tailwind CSS v3 + `tailwindcss-animate` |
| Fonts | Plus Jakarta Sans (body), Amiri (Arabic) |
| Forms | react-hook-form + zod + @hookform/resolvers |
| Animations | Framer Motion |
| Date handling | date-fns v3 |
| Charts | Recharts |
| QR codes | qrcode.react |
| Notifications | sonner + shadcn toast |
| Deployment | FTP via GitHub Actions → `masdik.iqis.sch.id` |
| Lint | ESLint 9 + typescript-eslint |

---

## 3. Project Structure

```
masdik-iqis-digital-hub/
├── .env                          # Supabase credentials (VITE_SUPABASE_URL, VITE_SUPABASE_PUBLISHABLE_KEY)
├── .github/workflows/main.yml    # CI/CD: build + FTP deploy on push to main
├── vercel.json                   # SPA rewrite rules (fallback to index.html)
├── vite.config.ts
├── tailwind.config.ts            # Custom colors (gold, emerald), fonts, animations
├── components.json               # shadcn/ui config
├── tsconfig.json / tsconfig.app.json / tsconfig.node.json
├── supabase/
│   ├── config.toml               # project_id = ozwshtyhnbayieximrkh
│   └── migrations/               # 14 SQL migration files (schema history)
├── public/
│   ├── favicon.png
│   ├── robots.txt
│   └── sitemap.xml
└── src/
    ├── main.tsx                  # React root mount
    ├── App.tsx                   # Router + providers + AdminRoute guard
    ├── index.css                 # Tailwind + CSS vars + Islamic custom utilities
    ├── App.css
    ├── vite-env.d.ts
    ├── assets/
    │   ├── logo-masjid.png       # Color logo (used in Navbar)
    │   ├── logo-masjid-white.png # White logo variant
    │   └── qris-donasi.jpg       # QRIS donation QR image (BSI 7301136287)
    ├── types/
    │   └── index.ts              # Shared TS interfaces: PrayerTime, Event, BookingRequest, DKMMember, Transaction
    ├── hooks/
    │   ├── useAuth.tsx           # AuthContext: user, session, isAdmin, signIn/signUp/signOut
    │   ├── use-toast.ts          # Toast hook (shadcn)
    │   └── use-mobile.tsx        # Mobile breakpoint hook
    ├── lib/
    │   └── utils.ts              # cn() helper (clsx + tailwind-merge)
    ├── integrations/supabase/
    │   ├── client.ts             # createClient<Database>(...) singleton
    │   └── types.ts              # Auto-generated Supabase types (full DB schema)
    ├── pages/
    │   ├── Index.tsx             # Home: ArabicGreeting + PrayerTimes + QuickLinks + About
    │   ├── Kegiatan.tsx          # Activity calendar page
    │   ├── ETaklim.tsx           # E-learning page (Coming Soon)
    │   ├── Pustaka.tsx           # Library page (public browsing)
    │   ├── Layanan.tsx           # Reservation page (public form)
    │   ├── Struktur.tsx          # DKM org structure page
    │   ├── Saldo.tsx             # Financial info page
    │   ├── Admin.tsx             # Full admin dashboard (tabs: Kegiatan, Reservasi, Keuangan, Pustaka, Absensi)
    │   ├── AdminLogin.tsx        # Email/password login form
    │   ├── AttendanceManagement.tsx  # Per-activity QR + attendance records (admin)
    │   ├── AttendanceForm.tsx    # Public QR-scan attendance form (/absen/:token)
    │   └── NotFound.tsx          # 404 page
    ├── components/
    │   ├── Navbar.tsx            # Fixed nav with mobile menu, active link highlighting
    │   ├── Footer.tsx            # Footer with links/info
    │   ├── ArabicGreeting.tsx    # Bismillah / Assalamu'alaikum display
    │   ├── PrayerTimes.tsx       # Live clock + prayer times card (aladhan API)
    │   ├── EventCalendar.tsx     # Calendar + event list for Kegiatan page
    │   ├── BookingForm.tsx       # Reservation form with conflict detection
    │   ├── SaldoSection.tsx      # Balance display + QRIS card + transaction list
    │   ├── DKMStructure.tsx      # Hardcoded org chart (static data)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kdm-developers/masdik-iqis](https://github.com/kdm-developers/masdik-iqis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
