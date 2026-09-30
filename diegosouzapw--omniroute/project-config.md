---
trigger: always_on
description: 🌐 **Languages:** 🇺🇸 [English](../../../AGENTS.md)
---

# OmniRoute agent guide (Bosanski)

🌐 **Languages:** 🇺🇸 [English](../../../AGENTS.md)

---

> **Jedinstveni izvor istine.** Ovaj fajl sadrži SVA pravila projekta, konvencije, bilješke o arhitekturi
> i Stroga Pravila za svakog AI asistenta koji radi u ovom repozitoriju (Claude Code, Gemini, Codex,
> Copilot i bilo koji drugi agent). `CLAUDE.md` i `GEMINI.md` samo dodaju specifične razlike za asistenta
> i upućuju natrag ovdje. Kada pravilo treba promijeniti, promijenite ga OVDJE — nikada ga nemojte ponovo forkovati u
> fajl specifičan za asistenta.

## Brzi start

```bash
npm install                    # Instaliraj zavisnosti (automatski generiše .env iz .env.example)
npm run dev                    # Dev server na http://localhost:20128
npm run build                  # Production build (Next.js 16 standalone)
npm run build:release          # Release build
npm run lint                   # ESLint (očekivano 0 grešaka; upozorenja su već postojeća)
npm run typecheck:core         # TypeScript provjera (treba biti čista)
npm run typecheck:noimplicit:core  # Stroga provjera (bez implicitnog any)
npm run test:coverage          # Unit testovi + coverage gate (60/60/60/60 — statements/lines/functions/branches)
npm run check                  # lint + test kombinovano
npm run check:cycles           # Detekcija kružnih zavisnosti
npm run check:docs-all         # Pokreni nakon izmjene dokumentacije (uključuje fabricated-docs validaciju)
```

### Pokretanje testova

Prvo pokrenite najprecizniji test za izmijenjeni kod:

```bash
# Pojedinačni test fajl (Node.js native test runner — većina testova)
node --import tsx/esm --test tests/unit/your-file.test.ts

# Vitest (MCP server, autoCombo, cache)
npm run test:vitest

# Svi suite-ovi
npm run test:all
```

Ostali suite-ovi: `npm run test:e2e`, `npm run test:protocols:e2e`, `npm run test:ecosystem`.

Za punu matricu testova, pogledajte `CONTRIBUTING.md` → "Running Tests". Za duboku arhitekturu, pogledajte sekcije
Repository map i Reference Documentation u nastavku.

---

## Projekt na prvi pogled

**OmniRoute** — jedinstveni AI proxy/router. Jedna krajnja tačka (endpoint), 359 LLM provajdera, auto-fallback.

| Sloj          | Lokacija                | Svrha                                                                                                                                                                     |
| ------------- | ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| API Routes    | `src/app/api/v1/`       | Next.js App Router — ulazne tačke                                                                                                                                         |
| Handlers      | `open-sse/handlers/`    | Obrada zahtjeva (chat, embeddings, itd)                                                                                                                                   |
| Executors     | `open-sse/executors/`   | Provider-specifični HTTP dispatch                                                                                                                                         |
| Translators   | `open-sse/translator/`  | Konverzija formata (OpenAI↔Claude↔Gemini)                                                                                                                                 |
| Transformer   | `open-sse/transformer/` | Responses API ↔ Chat Completions                                                                                                                                          |
| Services      | `open-sse/services/`    | Combo rutiranje, rate limiti, keširanje, itd                                                                                                                              |
| Database      | `src/lib/db/`           | SQLite domenski moduli (176 migracija)                                                                                                                                    |
| Domain/Policy | `src/domain/`           | Policy engine, pravila troškova, fallback logika                                                                                                                          |
| MCP Server    | `open-sse/mcp-server/`  | 110 alata (45 kanonskih + memory/skill/GitHub/pool/gamification/plugin/Notion/Obsidian/local-corpus/RTK moduli), 3 transporta (stdio / SSE / Streamable HTTP), 33 scope-a |
| A2A Server    | `src/lib/a2a/`          | JSON-RPC 2.0 agent protokol                                                                                                                                               |
| Skills        | `src/lib/skills/`       | Proširivi framework vještina                                                                                                                                              |
| Memory        | `src/lib/memory/`       | Persistent konverzacijska memorija                                                                                                                                        |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [diegosouzapw/OmniRoute](https://github.com/diegosouzapw/OmniRoute) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
