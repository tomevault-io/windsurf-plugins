---
trigger: always_on
description: Tento soubor poskytuje konkrétní, okamžitě použitelné instrukce pro AI coding agenty a lidské přispěvatele pracující v tomto repozitáři. Pokrývá setup, vývojové workflow, testování a validační kroky specifické pro tento projekt.
---

# AGENTS.md

## Účel

Tento soubor poskytuje konkrétní, okamžitě použitelné instrukce pro AI coding agenty a lidské přispěvatele pracující v tomto repozitáři. Pokrývá setup, vývojové workflow, testování a validační kroky specifické pro tento projekt.

> **Pravidlo:** Pokud přidáš novou funkci, endpoint nebo změníš strukturu projektu, aktualizuj příslušný AGENTS.md (kořenový, `frontend/AGENTS.md` nebo `backend/AGENTS.md`).

---

## Popis projektu

**Automatizace updatu investičního čtvrtletníku** — webová aplikace pro AI-asistovanou tvorbu a správu investičního čtvrtletníku. Analytici zadávají instrukce a podkladové dokumenty pro každou z 8 kapitol; AI (OpenAI GPT-4o) generuje textové návrhy, které editor reviduje a schvaluje. Finální čtvrtletník se exportuje do PDF nebo Word.

---

## Mapa repozitáře

```
/
├── frontend/               # React 18 + Vite + TypeScript + Tailwind CSS
│   ├── src/
│   │   ├── pages/          # Stránky aplikace (Dashboard, ChapterEditor, ...)
│   │   ├── components/     # Znovupoužitelné UI komponenty
│   │   ├── hooks/          # Custom React hooks (useEditions, useChapter, ...)
│   │   ├── lib/            # API klient (Axios), utility
│   │   └── types/          # TypeScript typy (api.ts, domain.ts)
│   ├── .env.example        # Šablona env proměnných frontendu
│   └── AGENTS.md           # Instrukce specifické pro frontend
│
├── backend/                # NestJS + TypeScript + Prisma ORM
│   ├── src/
│   │   ├── auth/           # JWT autentizace, guards, strategie
│   │   ├── editions/       # CRUD pro vydání čtvrtletníku
│   │   ├── chapters/       # Správa kapitol, workflow schválení
│   │   ├── documents/      # Upload a parsování podkladových dokumentů
│   │   ├── ai/             # OpenAI integrace, prompt management
│   │   ├── export/         # PDF a DOCX export
│   │   ├── prisma/         # PrismaService (NestJS modul)
│   │   └── common/         # Guards, filters, interceptors, decorators
│   ├── prisma/
│   │   ├── schema.prisma   # Databázové schema
│   │   ├── migrations/     # Prisma migrace
│   │   └── seed.ts         # Seed data pro vývoj
│   ├── uploads/            # Nahrané podkladové dokumenty (NIKDY necommitovat)
│   ├── .env.example        # Šablona env proměnných backendu
│   └── AGENTS.md           # Instrukce specifické pro backend
│
├── .gitignore
├── README.md
└── AGENTS.md               # Tento soubor — přehled celého projektu
```

---

## Nasazené aplikace

- Název: `quarterly-reporting-v1.0`
- Lokální cesta: `~/apps/quarterly-reporting-v1.0`
- Git remote: `git@github.com:tvitovec1/quarterly-reporting-v1.0.git`
- Ověřená struktura po naklonování: `frontend/`, `backend/`
- Veřejná URL: `https://vitovec.aibr.cz`
- Backend API přes reverse proxy: `https://vitovec.aibr.cz/api`

---

## Quickstart

**Prerekvizity:**
- Node.js 20 LTS
- npm 9+
- PostgreSQL 15+ (lokálně nebo Docker)

```bash
# 1. Naklonuj repozitář a nainstaluj závislosti
git clone <repo-url>
cd <repo>

# 2. Backend — setup
cd backend
cp .env.example .env          # vyplň hodnoty (viz sekce Env proměnné)
npm install
npx prisma migrate dev        # vytvoří DB a spustí migrace
npx prisma db seed            # seed data: 3 uživatelé, 1 vzorové vydání
npm run start:dev             # backend běží na http://localhost:3000

# 3. Frontend — setup (nový terminál)
cd frontend
cp .env.example .env
npm install
npm run dev                   # frontend běží na http://localhost:5173
```

---

## Časté příkazy

### Instalace
```bash
cd backend && npm install
cd frontend && npm install
```

### Docker
```bash
# Zkopíruj šablonu proměnných pro Docker Compose
cp .env.example .env

# Produkční reverse proxy a TLS
docker compose up -d traefik

# Sestavení image bez spuštění kontejnerů
docker compose build

# Ověření konfigurace Compose
docker compose config
```

### Vývoj
```bash
# Backend
cd backend && npm run start:dev       # s hot-reload

# Frontend
cd frontend && npm run dev            # Vite dev server, port 5173
```

### Build
```bash
cd backend && npm run build           # TypeScript kompilace do dist/
cd frontend && npm run build          # Vite produkční build
```

### Lint / Formát
```bash
cd backend && npm run lint            # ESLint
cd frontend && npm run lint           # ESLint + TypeScript

cd backend && npm run lint:fix        # auto-oprava
```

### Typecheck
```bash
cd backend && npx tsc --noEmit
cd frontend && npx tsc --noEmit
```

### Testy
```bash
# Backend — unit testy
cd backend && npm test

# Backend — konkrétní soubor
cd backend && npm test -- --testPathPattern=auth.service

# Frontend — unit testy (Vitest)
cd frontend && npm test
```

### Databáze
```bash
cd backend
npx prisma migrate dev                        # spustit pending migrace
npx prisma migrate dev --name add_field       # vytvořit novou migraci
npx prisma db seed                            # seed data
npx prisma studio                             # GUI prohlížeč databáze
npx prisma generate                           # regenerovat Prisma klienta
```

---

## Testování & TDD workflow

Následuj **Red → Green → Refactor**:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tvitovec1/quarterly-reporting-v1.0](https://github.com/tvitovec1/quarterly-reporting-v1.0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
