---
trigger: always_on
description: Guida operativa per Codex e altri agenti AI che lavorano su adOmnia.
---

# AGENTS.md

Guida operativa per Codex e altri agenti AI che lavorano su adOmnia.

Questo file e' intenzionalmente pratico: prima aiuta l'agente a capire il progetto, poi gli dice come modificarlo senza rompere le convenzioni locali.

---

## PRODUCT-FIRST PHILOSOPHY (PRIMARY DIRECTIVE)

**adOmnia is a production-grade desktop developer toolbox. This is NOT a theoretical exercise, a code challenge, or a boilerplate.**

The *primary goal* is a **professional, coherent, modern, and truly usable FINAL desktop PRODUCT**. Every decision must prioritize end-user experience and overall application quality.

### What We Are NOT Aiming For
- ❌ Perfect unit tests or excessively high code coverage
- ❌ Premature micro-optimizations
- ❌ "Academic" clean architecture at the expense of agility
- ❌ Isolated demos or proof-of-concepts

### Priority Hierarchy

| 🔴 HIGH PRIORITY (Product) | 🟢 LOW PRIORITY (Internal) |
|:----------------------------|:---------------------------|
| User Experience (UX) | Excessive, unnecessary unit tests |
| Real-world Integration | Premature abstractions |
| Fluidity & Responsiveness | Artificial layering (e.g. 900 layers) |
| Graphical Cohesion | Over-engineering solutions |
| Complete Workflows | |
| Perceived Stability | |
| User Journey | |
| Evolutionary Architecture | |
| Real Backend/Frontend Connection | |

**Rule:** when in doubt, bias toward what the user sees and touches. Internal code quality matters only insofar as it enables a better product.

---

## Current Project Status (HONEST ASSESSMENT)

Agents must understand the *real, current state* of the project:

- **Current runtime is Wails 2 + Go backend + React 18/TypeScript frontend.**
- Some older documentation may still mention Tauri/Rust or legacy paths; treat the actual files in this repo as the source of truth.
- **Backend is significantly more advanced** than the frontend
- **Frontend is still incomplete and nascent**
- **Many backend features are not yet connected** to the frontend
- **The UI is not yet professional** — lacks visual cohesion
- **Many modules still resemble mocks or prototypes**
- **Some features are technically functional but do NOT yet deliver a final product experience**

**Implication:** the highest-impact work is closing the frontend/backend gap, achieving visual cohesion, and elevating the UI to production quality — NOT adding more backend features in isolation.

---

## Canonical Project Context

Before planning or changing non-trivial behavior, use these docs to understand the product quickly:

| File | Use it for |
|------|------------|
| `docs/SOUL.md` | Product soul, UX philosophy, long-term vision, and what adOmnia should feel like as a finished desktop product. |
| `docs/funzionalita.md` | Fast inventory of all major features and modules. Read this when you need to understand what already exists before adding or moving functionality. |
| `docs/adomnia-roadmap-checkbox.md` | Roadmap and completion checklist across major product areas. |
| `docs/TODO.md` | Active bugs, gaps, and near-term work queue. |
| `README.md` | Public-facing product positioning and build overview. |

**Rule:** if you are unsure whether a feature already exists, first search the code, then check `docs/funzionalita.md`. If you are unsure how a feature should feel or fit the product, check `docs/SOUL.md`.

---

## Identita del Progetto

**adOmnia** è un desktop API development toolbox costruito attorno a **quattro pilastri difendibili:**

1. **Local-First** — niente cloud, niente account, niente telemetria. I dati non lasciano mai la macchina.
2. **User-Extensible** — plugin system, skin importabili, template condivisibili, workspace versionabili.
3. **Browser Debugging Integrato** — debug di pagine web esterne dentro lo stesso tool per API testing (nessun competitor lo fa).
4. **Enterprise & Legacy First-Class** — SOAP, WSDL, WS-Security, mTLS, JKS, eIDAS, Berlin Group.

**Stack tecnologico:**
- **Frontend:** React 18 + TypeScript + Vite
- **Desktop shell:** Wails 2
- **Backend:** Go
- **State frontend:** Zustand + React hooks
- **Persistence:** bbolt, local files, local workspaces, settings bindings
- **Distribuzione:** Eseguibile portabile singolo (`.exe`, `.app`, binario Linux)
- **Filosofia:** Local-first, privacy-first, user-extensible

---

## Prima di Toccare Codice

1. **Leggi CLAUDE.md** per architettura dettagliata, pattern, e binding Wails/Go
2. **Leggi `docs/SOUL.md`** se il cambio tocca UX, posizionamento prodotto, visual design o comportamento percepito
3. **Consulta `docs/funzionalita.md`** per capire velocemente tutte le funzionalità già presenti e non duplicare moduli esistenti
4. **Controlla il file specifico più vicino alla feature** — questo progetto preferisce cambi piccoli e localizzati
5. **Mantieni la filosofia local-first:** qualunque dato deve stare in storage locale, bbolt, localStorage o essere esportabile come file
6. **Se tocchi storage, settings o workspace `.adomnia`**, preserva backward compatibility e documenta la migrazione
7. **Ogni feature deve allinearsi con almeno 2 dei 4 pilastri** — vedi "Four Pillar Decision Framework" in CLAUDE.md

---

## Struttura Reale del Repo


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Andrea-Cavallo/adOmnia](https://github.com/Andrea-Cavallo/adOmnia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
