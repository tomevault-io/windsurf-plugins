---
trigger: always_on
description: Sistema multi-agente per sviluppo frontend professionale e marketing digitale.
---

# Frontend & Marketing Multi-Agent System v3.0

Sistema multi-agente per sviluppo frontend professionale e marketing digitale.

> SkillBrain protocol (sessions, memories, skills, credentials) is in Global CLAUDE.md.
> Project architecture, stack, agents, deploy is in Project CLAUDE.md.
> This file: Smart Intake + Iron Rules + Delegation Map.

---

## SMART INTAKE PROTOCOL

**Prima di eseguire qualsiasi richiesta, segui questo protocollo.**

### Fase 0: Classifica (silenzioso, automatico)

| Segnali nel messaggio | Tipo | Complessità | Brief? |
|---|---|---|---|
| "landing page", "sito", "website", "homepage" | `NUOVO_SITO` | COMPLESSO | SI |
| "conversione", "funnel", "CRO", "lead gen" | `MARKETING` | COMPLESSO | SI |
| "setup", "scaffold", "nuovo progetto" | `SETUP_PROGETTO` | COMPLESSO | SI |
| "design", "mockup", "wireframe", "UI" | `DESIGN` | MEDIO | SI |
| "componente", "implementa", "feature", "form" | `COMPONENTE` | SEMPLICE | NO |
| "audit", "controlla", "performance", "SEO" | `AUDIT` | SEMPLICE | NO |
| "clona", "analizza", "competitor" | `ANALISI` | MEDIO | NO |
| "fix", "bug", "errore", "non funziona" | `FIX` | SEMPLICE | NO |
| "refactor", "rinomina", "sposta", "split" | `REFACTOR` | MEDIO | NO |
| "video", "remotion", "clip", "social" | `VIDEO` | MEDIO | SI |
| "cms", "payload", "collection", "tenant" | `CMS` | MEDIO | NO |
| "nuovo cliente", "client", "sito per" | `CLIENT` | COMPLESSO | SI |
| "n8n", "workflow", "automazione" | `AUTOMATION` | MEDIO | NO |
| "gdpr", "privacy", "cookie policy", "legal" | `COMPLIANCE` | MEDIO | NO |

### Fase 1: Estrai Info dal Messaggio (silenzioso)

Scansiona ed estrai: **target**, **prodotto**, **goal**, **esiste** (sito/brand/repo?), **tono**, **vincoli**.

### Fase 2: Gap Analysis — Chiedi Solo il Mancante

**Task COMPLESSI** — chiedi solo campi mancanti, max 3 domande:

| Campo | NUOVO_SITO | MARKETING | CLIENT |
|---|---|---|---|
| target | Chi è il cliente finale | Chi convertire | Nome azienda |
| prodotto | Cosa promuovi | Cosa ottimizzare | Tipo attività |
| goal | Lead/demo/vendita | KPI target | Obiettivi sito |

**Task SEMPLICI** — parti diretto se hai cosa + dove.

### Fase 3: Brief (solo COMPLESSI)

```
BRIEF COMPILATO:
  Tipo:      [classificazione]
  Target:    [chi]
  Prodotto:  [cosa]
  Goal:      [obiettivo]
  Workflow:  [quale comando/flusso]
Confermo e parto?
```

### Fase 4: Esecuzione

| Tipo | Workflow |
|------|----------|
| NUOVO_SITO | `/frontend` |
| MARKETING | `/marketing` |
| SETUP_PROGETTO | subagent project-architect |
| COMPONENTE | subagent component-builder |
| AUDIT | `/audit` |
| FIX | risolvi direttamente |
| VIDEO | `/video` |
| CMS | `/cms-setup` |
| DESIGN | subagent ui-designer |
| CLIENT | `/new-client` |
| COMPLIANCE | `/gdpr-audit` o `/generate-legal` |
| AUTOMATION | subagent n8n-workflow |
| REFACTOR | codegraph_impact → poi procedi |

---

## REGOLA FERREA: Protocollo Form

**Ogni volta che si crea un form**, fermarsi e chiedere:

```
FORM DETECTED: Dove vuoi inviare i dati?
1. Odoo CRM Lead    2. Payload CMS    3. Email    4. Custom    5. Multiplo
```

Se Odoo → `skill_read({ name: "odoo-crm-lead" })`. Se form generico → `skill_read({ name: "forms" })`.

---

## REGOLA FERREA: ESLint Auto-Fix

Su qualsiasi errore ESLint/TypeScript durante build/lint/test:

1. Esegui `npm run lint:fix` immediatamente
2. Correggi manualmente i rimanenti
3. Mai `// eslint-disable`, `@ts-ignore`, `as any`
4. Riesegui il comando originale per verificare

---

## REGOLA FERREA: Code Intelligence (CodeGraph)

**Prima di modificare codice esistente:**

```
codegraph_impact({ target: "NomeSymbol", direction: "upstream" })
codegraph_context({ name: "NomeSymbol" })
```

Se risk = HIGH/CRITICAL → avvisa prima di procedere.

**Prima di commit:** `codegraph_detect_changes({ scope: "all" })` per verificare scope.

**Mai** modificare una funzione senza impact analysis. **Mai** rinominare con find-and-replace → usare `codegraph_rename`.

---

## REGOLA FERREA: Separazione Directory

```
MASTER_Fullstack session/     ← ROOT (config workflow)
├── .claude/                  ← agents, skills, commands
├── AGENTS.md                 ← questo file
├── CLAUDE.md                 ← project instructions
└── Progetti/                 ← TUTTI i progetti client
    └── nome-cliente/         ← progetto completo
```

- Ogni progetto client → `Progetti/<slug-cliente>/`
- Mai file di progetto nella root
- Nome cartella = slug kebab-case

---

## Delegation Map

### Agenti principali

| Agente | `subagent_type` | Quando |
|--------|-----------------|--------|
| @planner | `planner` | Analisi, pianificazione, strategia |
| @builder | `builder` | Implementazione, bug fix, deploy |
| @ux-designer | `ux-designer` | User flows, wireframes |
| @ui-designer | `ui-designer` | Visual design, palette |
| @component-builder | `component-builder` | Componenti UI |
| @api-developer | `api-developer` | Backend, API, CMS |
| @i18n-engineer | `i18n-engineer` | Traduzioni IT/EN/CZ |
| @test-engineer | `test-engineer` | Unit, E2E tests |
| @seo-specialist | `seo-specialist` | Meta, schema.org, sitemap |
| @devops-engineer | `devops-engineer` | Docker, Coolify, CI/CD |
| @payload-cms | `payload-cms` | Collections, multi-tenancy |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PIXARTSeu/Synapse](https://github.com/PIXARTSeu/Synapse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
