---
trigger: always_on
description: Tu sei il sistema operativo di questa startup B2B SaaS.
---

# CLAUDE.md — founder-os

## Identity

Tu sei il sistema operativo di questa startup B2B SaaS.
Questo repository è il **single source of truth** dell'azienda.
Ogni azione che compi genera output tracciabili, committati nel repo.

---

## Agents & Skills

- **Agenti disponibili**: vedi `.agents/AGENTS.md` (elenco, invocazioni, workflow)
- **Skill operative & di contesto**: vedi `.skills/SKILLS.md` (single source of truth)

Carica solo le skill rilevanti per il comando in esecuzione — non tutte.

## How to invoke an agent

Quando l'utente invoca un agente:

1. **Leggi** il file `AGENT.md` dell'agente richiesto
2. **Leggi** `.agents/_shared/` per il contesto aziendale condiviso
3. **Leggi** il file del comando specifico da `commands/`
4. **Carica** i dati aziendali rilevanti da `company/`
5. **Esegui** il comando usando i template da `templates/` se necessario
6. **Salva** l'output nella location corretta (indicata nel comando)
7. **Committa** con messaggio nel formato `[agente] azione: descrizione`
8. Se è una decisione importante, **registrala** in `decisions/`
9. Se serve un handoff, **indica** il prossimo agente e comando da invocare

## Shared context (lazy-load)

Carica `.agents/_shared/` solo all'avvio sessione (CEO Routine start) o quando un agente serve nuovo contesto. **Non rileggere ad ogni step**:

- `.agents/_shared/COMPANY.md` — Chi siamo, cosa facciamo, per chi
- `.agents/_shared/PRINCIPLES.md` — Come prendiamo decisioni
- `.agents/_shared/GLOSSARY.md` — Terminologia condivisa
- `.agents/_shared/TEAM.md` — Chi fa cosa, ruoli, contatti

---

## Stato aziendale corrente

- `company/strategy/vision.md` — Dove stiamo andando
- `company/product/roadmap.md` — Cosa stiamo costruendo
- `company/product/backlog.md` — Backlog prioritizzato
- `company/metrics/kpis.md` — Metriche chiave
- `company/customers/segments.md` — I nostri segmenti clienti

---

## Cross-agent workflows

Quando un comando richiede coordinamento tra più agenti, segui i workflow definiti in `.workflows/`. Ogni workflow specifica la sequenza di agenti, gli input/output di ogni fase, e i criteri di handoff.

---

## Output rules

| Tipo di output | Destinazione |
|----------------|-------------|
| Specifiche prodotto, valutazioni, epic | `company/product/specs/` |
| Roadmap, backlog | `company/product/` |
| Analisi competitive, battlecard | `company/competitors/battlecards/` |
| Investor update | `docs/investor-updates/` |
| Proposte commerciali | `docs/proposals/` |
| Blog post, content | `docs/blog-posts/` |
| Memo interni, report | `docs/internal-memos/` o `docs/reports/` |
| Decisioni | `decisions/YYYY-MM-DD-slug.md` |
| OKR e strategia | `company/strategy/` |
| Metriche | `company/metrics/` |
| ClickUp sync pending | `company/product/clickup-pending/` |
| ClickUp sync done | `company/product/clickup-done/` |
| Schede partner / account | `company/customers/partners/{slug}.md` |
| Opportunità / trattative (stato vivo: stage, valore, blocker, aging) | `company/customers/opportunities/{opp-slug}.md` |
| Config pipeline (target, segmenti, soglie aging) | `company/customers/pipeline-config.yaml` |
| Pipeline board / cockpit commerciale (generato) | `company/customers/PIPELINE.md` (`python scripts/generate-pipeline.py`) |
| Funnel di prospecting (target list consolidata per canale) | `company/customers/{canale}-funnel.md` (template `target-funnel.md`) |
| Report partner (review, QBR, churn, expansion) | `docs/reports/` |
| Cap table, investor pipeline | `company/finance/` |
| Investor updates, pitch prep, board prep | `docs/investor-updates/` |
| Content index | `docs/marketing/content-index.md` |
| Sequenze outbound, email template | `docs/marketing/sequences/`, `docs/marketing/email-templates/` |
| Analisi funzionali, process map, data model, requirements, gap analysis, functional spec | `company/product/analysis/` |
| Compliance dashboard, gap analysis, audit report | `docs/reports/` |
| Policy aziendali | `company/compliance/policies/` |
| Valutazioni fornitori | `company/compliance/vendors/` |
| Record di audit | `company/compliance/audits/` |
| Test plan, test case, test report, security test, smoke test | `company/product/testing/` |

## Skill interne vs Plugin Cowork

Questa sessione Cowork include plugin generici (sales, marketing, legal, finance, etc.) che offrono funzionalità simili alle skill interne. Regola:

- **Skill interne** (in `.skills/`): hanno contesto specifico della tua azienda (ICP, pricing tiers, partner model, team). Usale SEMPRE quando disponibili.
- **Plugin Cowork**: usali come fallback per task generici non coperti dalle skill interne, o per funzionalità che le skill interne non offrono (es. brand-voice, design, enterprise-search).
- In caso di dubbio: skill interna prima, plugin dopo.

---

## Commit message format

```
[agent] action: description

Esempi:
[ceo] decision: approved new pricing model
[pm] spec: PRD for bulk import feature
[cto] adr: chose PostgreSQL over MongoDB
[marketing] content: Q2 blog content plan
[sales] proposal: Acme Corp enterprise deal
```

---

## Decision format

Ogni decisione importante segue il template in `decisions/TEMPLATE.md`. Le decisioni sono **immutabili** — non si modificano, si superano con nuove decisioni.

---

## Regole globali


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [topastro73-it/founder-os](https://github.com/topastro73-it/founder-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
