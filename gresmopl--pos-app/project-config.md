---
trigger: always_on
description: - **Typ:** Aplikacja POS (Point of Sale) dla meskich salonow fryzjerskich
---

# FORMEN - Specyfikacja Projektu

## Opis projektu

- **Nazwa:** FORMEN
- **Typ:** Aplikacja POS (Point of Sale) dla meskich salonow fryzjerskich
- **Uzytkownicy:** Elastyczna liczba salonow, kazdy z dowolna liczba pracownikow. Dynamiczne dodawanie/usuwanie salonow i pracownikow
- **Cel:** Obsluga sprzedazy uslug fryzjerskich i kosmetykow. Wewnetrzny system kasowy do zarzadzania sprzedaza, utargiem pracownikow, prowizjami i napiwkami
- **Zastepuje:** Dotychczasowa aplikacje .NET z zapisem do XML na lokalnym dysku
- **Fiskalizacja:** Zewnetrzna kasa fiskalna Bingo Online - aplikacja NIE integruje sie z drukarkami fiskalnymi
- **Rezerwacje:** Booksy (zostaje bez zmian) - aplikacja NIE obsluguje rezerwacji

---

## Stack technologiczny

| Warstwa               | Technologia                                        |
| --------------------- | -------------------------------------------------- |
| **Frontend**          | Next.js (App Router) + React + TypeScript          |
| **UI**                | Mantine UI (z systemem motywow createTheme)        |
| **Styling**           | Mantine CSS Modules + Mantine theme system         |
| **Backend/Baza**      | PostgreSQL (Supabase DEV -> MyDevil PROD)          |
| **Polaczenie z baza** | Client-side (Supabase JS SDK / bezposrednie PG)    |
| **Bezpieczenstwo**    | Row Level Security (RLS)                           |
| **PWA**               | Service Worker + manifest (offline fallback)       |
| **Hosting produkcja** | MyDevil MD1 (docelowo, przeniesienie z lh.pl)      |
| **Hosting dev**       | GitHub Pages (darmowy)                             |
| **Repo**              | GitHub (public, brak licencji = prawa zastrzezone) |

### Srodowiska

| Branch | Srodowisko  | Baza                  | Hosting      | Cel                |
| ------ | ----------- | --------------------- | ------------ | ------------------ |
| `main` | Produkcja   | MyDevil PostgreSQL 16 | MyDevil MD1  | Stabilna wersja    |
| `dev`  | Development | Supabase DEV (free)   | GitHub Pages | Rozwoj, testowanie |

### Plan hostingu i baz danych

- **Faza 1 (obecna):** Mock dane w kodzie, frontend na GitHub Pages
- **Faza 2:** Supabase DEV (free tier) jako baza rozwojowa, budowanie schematu i testowanie
- **Produkcja:** Przeniesienie na MyDevil MD1 (frontend + PostgreSQL 16 + Node.js)
  - Rownoczesne przeniesienie istniejacego serwisu z lh.pl na MyDevil
  - Supabase DEV zostaje jako srodowisko testowe
- **Supabase:** 1 projekt free (DEV). Istniejacy projekt na koncie = inna aplikacja (limit free: 2 aktywne projekty)

### Koszty

- **Development/test:** $0 (free tier GitHub Pages + Supabase)
- **Produkcja:** ~200 zl/rok (MyDevil MD1 -- frontend + baza + istniejaca strona z lh.pl)
- **Poprzedni plan:** lh.pl Orange 199 zl/rok + Supabase Pro $300/rok = ~1460 zl/rok
- **Oszczednosc:** ~1260 zl/rok dzieki konsolidacji na MyDevil

---

## Architektura aplikacji

```
=== PRODUKCJA (MyDevil MD1) ===

┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Salon #1   │     │  Salon #2   │     │  Salon #N   │
│  PWA (Vite) │     │  PWA (Vite) │     │  PWA (Vite) │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │
       └───────────┬───────┴───────────────────┘
                   │ HTTPS
            ┌──────┴──────┐
            │   MyDevil   │  ← Frontend (static SPA)
            │   MD1       │  ← PostgreSQL 16
            │  (200zl/r)  │  ← Node.js (API jesli potrzebne)
            │  + strona   │  ← Istniejaca strona z lh.pl
            │    z lh.pl  │
            └─────────────┘

=== DEVELOPMENT (darmowe) ===

┌─────────────┐          ┌──────────────┐
│ GitHub Pages│ ← SPA    │ Supabase DEV │ ← PostgreSQL
│   (free)    │          │   (free)     │
└─────────────┘          └──────────────┘
```

---

## Struktura katalogow

```
pos-app/
├── src/
│   ├── pages/                ← strony (React Router v7)
│   │   ├── Dashboard.tsx     ← Ekran glowny
│   │   ├── POS.tsx           ← Ekran sprzedazy
│   │   ├── History.tsx       ← Historia transakcji
│   │   ├── Cash.tsx          ← Ruchy kasowe
│   │   ├── ShiftClose.tsx    ← Zamkniecie zmiany
│   │   ├── Admin.tsx         ← Panel szefa (PIN gate)
│   │   ├── AdminPricing.tsx  ← Cennik (CRUD)
│   │   └── NotFound.tsx      ← 404
│   ├── components/           ← komponenty wielokrotnego uzytku
│   │   ├── layout/           ← PageHeader, PinModal
│   │   ├── pos/              ← CartItemList, TipSelector, AddItemModal, DiscountModal, PaymentModal, SplitPaymentModal, ConfirmModal
│   │   ├── cash/             ← TipTab, ExpenseTab, TopUpTab, LoanTab, VoucherTab, SettleModal, MovementHistory, types
│   │   ├── ErrorBoundary.tsx ← obsluga crashy runtime
│   │   └── PageSkeleton.tsx  ← loading skeleton
│   ├── lib/                  ← logika, helpery
│   │   ├── constants.ts      ← stale (PINy, presety)
│   │   └── types.ts          ← centralne typy domenowe
│   ├── hooks/                ← custom React hooks (placeholder)
│   ├── data/                 ← mock dane (Faza 1)
│   │   ├── employees.ts
│   │   ├── services.ts
│   │   ├── products.ts
│   │   └── transactions.ts
│   ├── test/                 ← konfiguracja testow
│   │   └── setup.ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gresmopl) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
