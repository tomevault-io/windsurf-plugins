---
trigger: always_on
description: Ez egy modern, reszponzív **multi-termékes marketing weboldal** az **Arrabona** által fejlesztett két szoftverre:
---

# CLAUDE.md - Arrabona Digitalizálási és AI Chat Platform

## Projekt Áttekintés

Ez egy modern, reszponzív **multi-termékes marketing weboldal** az **Arrabona** által fejlesztett két szoftverre:

1. **Catalog+ v1.2** - Professzionális könyvtári digitalizálási és import szoftver
2. **Libra AI** - AI alapú dokumentum keresési és chat interfész

**Weboldal célja**: Marketing és lead generálás mindkét szoftverhez
**Célközönség**: Könyvtár adminisztrátorok, könyvtárosok, döntéshozók
**Nyelvek**: Magyar (alapértelmezett) és Angol
**Dizájn**: Világos mód, sötétkék (#1e3a8a) akcentszínnel

---

## Szoftveres Szolgáltatások Leírása

### 1. CATALOG+ v1.2 - Könyvtári Digitalizálási Platform

**Fő célja**: Professzionális könyvtári katalógus digitalizálása, adatkezelése és export

**Mit nyújt a szoftver?**
- **Katalóguskártyák digitalizálása**: Scannelt képekből és PDF-ekből AI alapú OCR-rel
- **Külső adatbázis importálása**: Más könyvtári rendszerekből származó adatok átvétele
- **Intelligens szövegfelismerés**: GPT-5 alapú OCR technológiával pontos adatkinyerés
- **Adatkezelés és tisztítás**: Automatikus és manuális adatfeldolgozás
- **Fuzzy search**: Intelligens keresés az összes feldolgozott dokumentumon
- **Standard formátumok exportálása**: Excel és USMARC formátumok

**Munkafolyamat (3 lépés)**:
1. **Importálás és Digitalizálás**: Scannelt képek, PDF, külső adatbázis (Arrabona csapata végzi az OCR feldolgozást)
2. **Böngészés és Keresés**: Digitalizált katalógus megtekintése, fuzzy matching keresés
3. **Exportálás**: Excel és USMARC formátumok, könyvtári kezelőrendszerekbe integráció

**Technikai Architektúra**: PySide6 GUI | Azure OpenAI GPT-5 OCR | SQLite/Supabase DB | Fuzzy matching | Excel/USMARC export

**Árazási Modell**:
- Kis (1-10k cédula): 100 Ft/cédula
- Közepes (10k-50k): 90 Ft/cédula (POPULÁRIS)
- Nagy (50k+): 80 Ft/cédula
- Kliens licensz: 30.000 HUF (egyszeri)

---

### 2. LIBRA AI - Intelligens Dokumentum Chat és Keresési Platform

**Fő célja**: Dokumentumokkal való beszélgetés, AI alapú keresés és információ kinyerés

**Mit nyújt a szoftver?**
- **Dokumentum feltöltés**: Scannelt képek, PDF-ek, szövegfájlok
- **AI Chat interfész**: Természetes nyelvű kérdések feltételére válaszok az adatbázisból
- **Intelligens keresés**: Vector-alapú szemantikus keresés
- **Forráskövetés**: Pontosan melyik dokumentumból származik az információ
- **RAG technológia**: Retrieval-Augmented Generation (nem hallucináció alapú)

**Munkafolyamat (3 lépés)**:
1. **Dokumentum Feldolgozás**: Feltöltés → Szöveg kinyerés → Chunking (512-1024 token) → Embedding (Gemini-001) → Vector DB
2. **Chat Felület**: Libra AI chat interfész elérhetővé válik
3. **Intelligens Keresés és Válaszadás**: Kérdés vektorizálása → Top-K chunk keresés → AI válasz (Gemini 2.5 Flash) → Forráskövetés

**Technikai Architektúra**: FastAPI | Google Gemini 2.5 Flash | Gemini-Embedding-001 | Supabase pgvector | React/Vue frontend

**Árazási Modell**:
- Ingyenes (0-100 oldal): 0 Ft
- Starter (101-5k oldal): 80 Ft/oldal
- Pro (5k-50k): 70 Ft/oldal (POPULÁRIS)
- Enterprise (50k+): 60 Ft/oldal

---

## Weboldalas Fájlstruktúra

```
arrabona_digitalizalas_weboldal/
├── index.html              # HOMEPAGE - Termékkiválasztó (2 kártya)
├── web_ocr/index.html      # Catalog+ marketing oldal
│   ├── js/lang_ocr_hun.js, lang_ocr_eng.js
│   └── img/product_*.png
├── web_chat/index.html     # Libra AI marketing oldal
│   ├── js/lang_chat_hun.js, lang_chat_eng.js
│   └── img/product_*.png
├── css/style.css           # OSZTOTT STÍLUS - Mindhárom oldalhoz
├── js/main.js              # OSZTOTT JAVASCRIPT - URL alapú nyelvfájl betöltés
├── js/lang_home_hun.js, lang_home_eng.js
└── img/logo.png, ocr.png, ai_chat.png
```

**Fájlstruktúra Logikája**:
- **Root (/)**: Homepage - termékkiválasztó oldal
- **web_ocr/**: Catalog+ specifikus oldal + erőforrások
- **web_chat/**: Libra AI specifikus oldal + erőforrások
- **css/ + js/ (root)**: Osztott stílus és logika mindhárom oldalhoz
- **img/ (root)**: Megosztott képek (logó, termékthumbnailok)

---

## Oldal Szekciók

### Homepage (index.html)
**Szerkezet**: Header + Hero (redukált) + 2 termékkártya + Footer
- `.hero:has(.hero-home)` - Redukált padding (30px)
- `.hero-home .hero-title` - 2.5rem; `.hero-home .hero-subtitle` - 1.1rem
- `.products-grid` - 2 oszlopú layout

### Catalog+ oldal (web_ocr/index.html)
**Szerkezet**: Header → Hero → Features (4 kártya) → Workflow (3 lépés Swiper) → Pricing (4 csomag + kliens licensz 30k Ft) → Contact → Footer

### Libra AI oldal (web_chat/index.html)
**Szerkezet**: Header → Hero → Intro → Features (3-4 kártya) → Workflow (3 lépés Swiper) → Pricing (3 csomag: Free, Starter 80 Ft/old, Pro 70 Ft/old*) → Pricing Note (Magyarázat az árról) → Contact → Footer

**FONTOS**: AI provider nevek (Google Gemini, Gemini-Embedding-001) **nem jelennek meg** a weboldalon!

---

## CSS & JavaScript

### CSS (style.css) - Osztott stílusfájl

**Szekciók**: Reset → CSS Variables → Layout → Header → Hero → Components → Responsive Breakpoints

**Szín Rendszer**:
- `--primary-blue`: #1e3a8a | `--dark-blue`: #1e40af | `--light-blue`: #3b82f6 | `--accent-blue`: #60a5fa

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xyadam/arrabona_digitalizalas_weboldal](https://github.com/xyadam/arrabona_digitalizalas_weboldal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
