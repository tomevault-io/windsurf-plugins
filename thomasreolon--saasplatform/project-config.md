---
trigger: always_on
description: Blueprint repo per SaaS basati su crediti e job asincroni. Focus sulla core feature di valore, non sull'infrastruttura.
---

# Job Based SaaS Blueprint

## Why
Blueprint repo per SaaS basati su crediti e job asincroni. Focus sulla core feature di valore, non sull'infrastruttura.
Stato attuale: side project dev-only. Codice billing commentato; crediti gratuiti: 100 al signup + 1 ricarica gratuita da 100 (l'admin `ADMIN_EMAIL` ricarica senza limiti). Anti multi-account: hash SHA-256 (peppered, `IP_HASH_SALT`) dell'IP di signup nel registro append-only `signup_ip_seen` — IP già visto = niente bonus né ricarica, e sopravvive alla cancellazione GDPR.

## What
**Stack:**
- **Landing**: Astro (zero-JS, SEO-first) — app standalone, servita separatamente (CDN in prod).
- **App**: React SPA (`apps/app`, react-router + TanStack Query). Buildata e **servita dal backend (monolite)** alla root della sua origin (dev: `:8000`; prod: `app.domain.com`), same-origin con l'API (niente CORS). In dev gira su Vite (`:3000`) con proxy `/api` → `:8000`.
- **Backend**: FastAPI (Python) — espone `/api/v1/*` e serve la SPA compilata da `static/app` (catch-all sulla root per il client routing; prefissi `api`/`health`/`openapi.json` riservati).
- **Auth**: Firebase Auth (ID Bearer token verified server-side via Admin SDK).
- **DB**: Postgres, SQLModel/SQLAlchemy, Alembic.
- **Job queue**: Postgres-as-queue (`SELECT FOR UPDATE SKIP LOCKED`).
- **File storage**: Google Cloud Storage per i file di input/output dei job (opzionale per job). In dev fake-gcs-server (stesso code path). I byte stanno su GCS, i metadati su Postgres (`artifacts`).
- **Billing**: Stripe (scaffolding commentato/non attivo, crediti gratis).

**Struttura:**
```
.
├── apps/
│   ├── web/              # Astro (landing, standalone)
│   ├── app/              # React SPA (servita dal backend sotto /app)
│   └── api/              # FastAPI (API /api/v1/* + serve la SPA → monolite)
├── jobs/                 # Worker Python (Cloud Run Jobs)
├── libs/shared/          # Core condiviso (db, billing, storage, catalog dei job)
│   └── src/shared/catalog/   # ← un file per job (definizione completa)
├── infra/                # Docker compose / config infra
└── CLAUDE.md
```

**Modello dati core:** `users`, `credit_transactions` (ledger append-only), `jobs` (stato + output), `artifacts` (file di input/output, byte su GCS), `audit_log`. **Il saldo crediti è sempre `SUM(credit_transactions.amount) WHERE user_id = ?` (mai cachato).**

**Componenti React riusabili (`apps/app/src/components/`):**
- `ArtifactPreview.tsx` — visualizza artefatti di output (immagini, file) di un job completato
- `FileField.tsx` — input per upload di file con gestione async e barra di progresso
- `Icons.tsx` — set di icone SVG inline (usa solo queste; non aggiungere librerie esterne)
- `Marketplace.tsx` — modal per abilitare/disabilitare gruppi di job
- `Notifications.tsx` — dropdown notifiche nella topbar
- `Topbar.tsx` — barra di navigazione superiore (breadcrumb, tema, lingua, notifiche)
Prima di creare un nuovo componente verifica sempre se ne esiste già uno riusabile qui.
Pagine principali: `JobsPage.tsx` (lista job + 3 tab per tipo), `JobView.tsx` (form nuova esecuzione), `SettingsPage.tsx`, `Sidebar.tsx`, `Login.tsx`.

**App SPA — cartelle e pattern (`apps/app/src/`):** la logica condivisa vive in `lib/` (una sola via per ogni cosa):
- `api.ts` — UNICO wrapper fetch (bearer token + gestione errori). Mai `fetch` grezzo o axios.
- `queries.ts` — hook TanStack Query (`useJobs`, `useJobTypes`, …): unico modo per leggere/mutare dati server.
- `types.ts` — specchio TS delle response API; tienilo allineato ai campi usati.
- `i18n.ts` — `t(key)` per le stringhe statiche dell'UI (mai hardcodare testo in JSX), `td(str)` per le stringhe dinamiche dal catalog (display_name/description/titoli di output).
- `groups.ts` — raggruppa i job per `group` e annida i `subgroup` (es. marketing > reels) nella Sidebar.
Form di input e pagina risultati sono **generati dallo schema**: non scrivere UI per-job. Lo stile sta in `styles/global.css` (design token + classi, no CSS-in-JS).

**Aggiungere un job: UN solo file.** Crea `libs/shared/src/shared/catalog/<tipo>.py` (copia da `_template.py` nella stessa cartella). Tutto sta lì dentro:
- **Schemi** input/output (Pydantic). Per input-file usa `file_field(accept=..., max_mb=..., multiple=...)`; per gli output usa `result_field(artifact=..., format="eur|duration|number|text", hide=...)` così la pagina risultati mostra stat curate invece del JSON (gli artifact PDF/video/immagine sono resi inline).
- **Metadati + handler** via il decoratore `@job(type=..., cost=..., group=..., subgroup=..., name=..., input=..., output=...)` sulla funzione `run(ctx, inp) -> Output` (`subgroup` opzionale = sotto-cartella in sidebar).
- **Costo in crediti**: `cost=usd(<costo vivo stimato in $>)` da `shared.pricing` (1 credito ≈ 1¢ venduto, copre $0.006 di costo → margine 40%; minimo 10 crediti/job) con un commento che documenta cosa pesa. Se il costo scala con l'input (es. pagine di un PDF): `cost_per_unit=usd_per_unit(...)` + `cost_unit={"it":...,"en":...}` + `count_units=<fn(inp, load)>` — le unità vengono misurate al submit, prima dell'addebito (vedi `pdf_translate.py`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thomasreolon/SaaSPlatform](https://github.com/thomasreolon/SaaSPlatform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
