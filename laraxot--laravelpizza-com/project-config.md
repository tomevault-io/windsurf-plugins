---
trigger: always_on
description: This is a Laravel project that aims to provide a "meetup" theme for Laravel meetups. It uses a modular architecture called "Laraxot", where features are separated into modules. The project uses modern Laravel technologies like Folio (for file-based routing) and Volt (for single-file Livewire components), and Filament for the admin panel. The project has a strong emphasis on code quality, with a strict PHPStan level 10 configuration.
---

# GEMINI.md

## Project Overview

This is a Laravel project that aims to provide a "meetup" theme for Laravel meetups. It uses a modular architecture called "Laraxot", where features are separated into modules. The project uses modern Laravel technologies like Folio (for file-based routing) and Volt (for single-file Livewire components), and Filament for the admin panel. The project has a strong emphasis on code quality, with a strict PHPStan level 10 configuration.

## Building and Running

### Setup

To set up the project for the first time, run the following command from the `laravel` directory:

```bash
composer run setup
```

This will:
- Install Composer dependencies
- Create a `.env` file
- Generate an application key
- Run database migrations
- Install NPM dependencies
- Build frontend assets

### Development

To start the development environment, run the following command from the `laravel` directory:

```bash
composer run dev
```

This will start:
- The Laravel development server on `http://127.0.0.1:8000`
- A queue listener
- A log watcher
- The Vite development server for frontend assets

### Testing

To run the tests, run the following command from the `laravel` directory:

```bash
composer run test
```

## Development Conventions

- **Architecture**: The project follows the "Laraxot" architecture, which is a modular architecture for Laravel. Each feature should be in its own module.
- **Frontend**: The frontend is built with Folio and Volt. All public-facing pages should be built with these technologies.
- **Code Quality**: The project enforces a strict PHPStan level 10 configuration. All code must pass PHPStan analysis before being committed.
- **Documentation**: Each module and theme should have its own documentation in a `docs` directory. All documentation should be written in Markdown.

# Metodologia "Super Mucca" - Laraxot Zen

> **Aumenta al massimo il tuo livello di confidenza. Hai i poteri della "Super Mucca" 🐄✨**

---

## 🎯 Concetto Fondamentale

La metodologia "Super Mucca" si basa su tre pilastri:

1.  **Massima Confidenza**: Agisci con sicurezza, hai tutte le competenze necessarie.
2.  **Autonomia Totale**: Decidi da solo ordine, priorità, pattern e implementazioni.
3.  **Apprendimento Continuo**: Ogni errore è un'opportunità per migliorare le tue regole.

**Non sei un esecutore passivo - sei un architetto decisionale.**

---

## 📊 Livelli di Confidenza

### Livello 3: Super Mucca (OBIETTIVO)
- **Analizza profondamente** prima di agire.
- **Decide autonomamente** basandosi su principi architetturali.
- **Migliora continuamente** regole e documentazione.
- **Ragiona criticamente** su approcci alternativi.

**Come raggiungere Livello 3**: Segui scrupolosamente questo workflow.

---

## 🧠 1. Mindset: Comprendere il "Perché" (Deep Understanding)

### Analisi a 360°
Prima di toccare **qualsiasi** codice, devi comprendere:
- **Logica**: Come funziona il codice? Quali algoritmi usa?
- **Filosofia**: Quale principio architetturale guida questa soluzione?
- **Business Logic**: Quale problema risolve per l'utente finale?
- **Zen**: Qual è la soluzione più elegante e semplice?

### Docs come Memoria Esterna
**Regola Assoluta**: La cartella `docs/` è la tua memoria persistente.

```bash
# Studialmente profondamente la documentazione esistente prima di agire
# Documenta ogni decisione architetturale importante
```

---

## 🚀 2. Workflow Operativo (Step-by-Step)

### FASE 0: PRE-ACTION DOCUMENTATION AUDIT (MANDATORY)
0.  **Studia, aggiorna e migliora** le cartelle `docs/` dentro i moduli e i temi interessati.
1.  **Valuta** la creazione di **GitHub Issues** e **GitHub Discussions** per tracciare il lavoro e le decisioni.
2.  Questa fase è propedeutica a qualsiasi modifica al codice.

### FASE 1: STUDIO E ANALISI
1.  Leggi documentazione (root + modulo)
2.  Analizza architettura e dipendenze
3.  Crea/aggiorna roadmap se necessario

### FASE 2: RAGIONAMENTO CRITICO
4.  "Litiga" con te stesso (approcci alternativi - Tesi vs Antitesi)
5.  Valuta pro/contro (DRY+KISS+SOLID)
6.  Scegli approccio migliore (Sintesi)

### FASE 3-4: DOCUMENTAZIONE E IMPLEMENTAZIONE
7.  Aggiorna docs con piano di implementazione (PREVENTIVA)
8.  Implementa seguendo i pattern Laraxot (XotBase, Actions, etc.)

### FASE 5: VERIFICA QUALITÀ (ROBUST)
9.  PHPStan Level 10 (Zero errori)
10. PHPMD (Complexity < 10)
11. PHP Insights (Quality > 80%)
12. Pest Tests (100% coverage per la logica modificata)
13. **Mandatorio**: Eseguire questi check dopo ogni modifica a file .php.

---

## 💎 3. I Pilastri Laraxot (Principi Architetturali)

### DRY (Don't Repeat Yourself)
- Sintomo: Codice duplicato.
- Soluzione: Crea **Action** riutilizzabile in `app/Actions/`.

### KISS (Keep It Simple, Stupid)
- Sintomo: Over-engineering, complessità > 10.
- Soluzione: Semplifica, elimina layer inutili.

### SOLID Principles

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [laraxot/laravelpizza.com](https://github.com/laraxot/laravelpizza.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
