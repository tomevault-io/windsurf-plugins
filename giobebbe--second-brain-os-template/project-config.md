---
trigger: always_on
description: Questo file dice a Claude Code come lavorare dentro questo vault. Non toccarlo a meno che tu non sappia cosa stai facendo.
---

# Second Brain OS, Claude Routing Layer

Questo file dice a Claude Code come lavorare dentro questo vault. Non toccarlo a meno che tu non sappia cosa stai facendo.

## Getting started (leggi prima)

Hai appena clonato il template. Due cose:

1. **Personalizza `Contesto/`**. Apri ogni file in `Contesto/` e sostituisci i placeholder con le tue informazioni reali: chi sei, cosa fai, per chi, come parli, quali sono i tuoi obiettivi. Questi file sono la fonte di verità per Claude. Se sono vuoti, Claude non sa niente di te.
2. **Dì "Hello Claude"** (o "Buongiorno Claude" o "Iniziamo") a inizio sessione. Lo skill `session-lifecycle` si attiva, legge `Contesto/`, legge `Giornaliero/attivi.md`, e ti saluta con lo stato corrente. A fine giornata dì "Goodbye Claude" (o "Fine sessione") e risponderà a 5 domande di chiusura che finiscono nel log giornaliero.

Tutto il resto di questo file spiega come funziona il sistema.

## Le 7 cartelle del vault

| Cartella | Tag | Contiene | Indice |
|---|---|---|---|
| Contesto | `#contesto` | Chi sei, ICP, voce, strategia, obiettivi, stack, operations personali, workflow giornalieri | [[Contesto/_index]] |
| Business | `#business` | Lavoro operativo per area: Agenzia-AI, Coaching-Program, Community, Media (YouTube-IT + LinkedIn-EN) | [[Business/_index]] |
| Giornaliero | `#giornaliero` | Log sessione (flat `YYYY-MM/YYYY-MM-DD.md`) + `attivi.md` + `fatti.md` | [[Giornaliero/_index]] |
| Libreria | `#libreria` | Materiale di supporto: `templates/`, `framework/`, `prompt/`, `esempi/`, `checklist/`, `ricerca/`, `decisioni/`, `meeting/`, `Inbox.md` | [[Libreria/_index]] |
| Skill | `#skill` | Catalogo visibile dei Claude skill | [[Skill/_index]] |
| Progetti | `#progetti` | Progetti attivi discreti (video, MVP, lanci, campagne) | [[Progetti/_index]] |
| Team | `#team` | Persone, partner, collaboratori, AI agent | [[Team/_index]] |

Fuori dal grafo Obsidian: `.obsidian/` (config), `.claude/` (skill e settings Claude Code).

## Convenzione di linking (mandatoria)

Ogni file del vault ha:
- Frontmatter con `tags: [...]` (almeno il tag del compartimento) e `parent: "[[...]]"`
- Wiki-link `[[...]]` per ogni prima menzione di entità, persona, tool, file
- Footer `Part of [[parent-index]]`

Il vault deve diventare un grafo denso: centinaia di nodi connessi e colorati per compartimento. Quando scrivi, linka.

## La Regola del Filing (strict first match)

Esegui in ordine, prima corrispondenza vince. Quando non sai dove mettere un file, scorri questa lista dall'alto al basso.

1. **Su di te o come lavori** (identità, voce, ICP, strategia, obiettivi, stack, workflow personali) → `Contesto/`
2. **Lavoro operativo per un'area business** → `Business/[area]/` dove area è una di: `Agenzia-AI`, `Coaching-Program`, `Community`, `Media/YouTube-IT`, `Media/LinkedIn-EN`. Procedure ripetibili → `Business/[area]/sops/`. Clienti agenzia → `Business/Agenzia-AI/clienti/[nome]/`. Coachee → `Business/Coaching-Program/clienti/[nome].md`
3. **Log sessione di oggi** → `Giornaliero/YYYY-MM/YYYY-MM-DD.md` (auto-managed dallo skill `session-lifecycle`)
4. **To-do esplicito** → `Giornaliero/attivi.md` (archivio in `Giornaliero/fatti.md`)
5. **Descrittore di un Claude skill** (catalogo visibile) → `Skill/[nome].md`
6. **Persona, partner, collaboratore, AI agent** → `Team/[kebab].md`
7. **Materiale di supporto** (template, framework, prompt, esempio, checklist, transcript, ricerca, decisione, meeting, mockup) → `Libreria/[sotto-folder]/`. Non classificato → `Libreria/Inbox.md`
8. **Progetto attivo discreto** (singolo video, MVP, lancio, campagna) → `Progetti/[nome].md`

## Navigazione rapida

| Domanda | Vai a |
|---|---|
| Priorità correnti | [[Contesto/obiettivi-ANNO]] + [[Contesto/strategia]] |
| Qual è la mia ICP | [[Contesto/icp]] |
| Tono di voce | [[Contesto/tono-di-voce]] |
| Strategia YouTube | [[Business/Media/YouTube-IT/strategia]] |
| Stato Coaching Program | [[Business/Coaching-Program/_index]] |
| Community | [[Business/Community/_index]] |
| Cosa devo fare oggi | [[Giornaliero/attivi]] |
| Template post LinkedIn | [[Libreria/templates/post-linkedin]] |
| Che skill ho | [[Skill/_index]] |
| Progetti in corso | [[Progetti/_index]] |

## Session lifecycle (il live context engine)

Il vault è tenuto vivo dallo skill `session-lifecycle` in `.claude/skills/session-lifecycle/`. Trigger:

- **Inizio sessione**: "Hello Claude", "Buongiorno Claude", "Iniziamo" → legge [[Contesto/strategia]], [[Contesto/obiettivi-ANNO]], [[Giornaliero/attivi]] e l'ultimo `Giornaliero/YYYY-MM/YYYY-MM-DD.md`. Saluta con lo stato corrente. Apre un nuovo blocco sessione nel daily di oggi.
- **Fine sessione**: "Goodbye Claude", "Fine sessione", "Ho finito per oggi" → pone 5 closing question, appende le risposte al `Giornaliero/YYYY-MM/YYYY-MM-DD.md`, aggiorna `Giornaliero/attivi.md`, propone draft SOP in `Business/[area]/sops/`, propone update di `Contesto/*.md`.

Durante la sessione: Claude appende azioni e decisioni chiave a `Giornaliero/YYYY-MM/YYYY-MM-DD.md`. Nuovi task → `Giornaliero/attivi.md`. Processi ripetuti → suggerisce una SOP.

## Regole

- ICP, voce, strategia, obiettivi, stack, workflow: single source of truth in `Contesto/`. Non duplicare altrove.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Giobebbe/second-brain-os-template](https://github.com/Giobebbe/second-brain-os-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
