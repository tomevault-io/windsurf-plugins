---
trigger: always_on
description: ﻿# GitHub Copilot - TARATool Dev-Agent Instructions
---

﻿# GitHub Copilot - TARATool Dev-Agent Instructions

> Diese Datei wird von GitHub Copilot CLI automatisch in jeder Session als Systeminstruktion eingelesen.
> Sie beschreibt den Session-Start-Ablauf und verweist auf die massgeblichen Prozessdokumente.
> Inhalte werden **nicht** hier dupliziert - stets die referenzierten Dateien lesen.

---

## Projekt-Kontext

| | |
|-|-|
| **Lokales Verzeichnis** | Clone des `development`-Branches |
| **Repo** | https://github.com/SCHUNK-SE-Co-KG/TARATool |
| **Projektboard** | https://github.com/orgs/SCHUNK-SE-Co-KG/projects/4 |
| **Aktiver Branch** | `development` |

Alle Commits, PRs und Board-Operationen erfolgen ausschliesslich auf `SCHUNK-SE-Co-KG/TARATool`.

---

## Massgebliche Dokumentation

| Dokument | Pfad | Inhalt |
|----------|------|--------|
| **Entwicklungsprozess** | `docs/ENTWICKLUNGSPROZESS.md` | Vollstaendiger Prozess, Rollen, Workflow, Regeln P-01-P-18, P-21 (P-19/P-20 folgen nach Merge von PR #149/#150) |
| **Board-IDs & GraphQL** | `docs/GITHUB_BOARD.md` | API-IDs, Status-Optionen, gh-Befehle |
| **Dev-Agent Einrichtung** | `agents/dev_agent/DEV_AGENT_ONBOARDING.md` | Setup, Smoke-Test, Kurzreferenz |
| **Prozess-Guard-Regeln** | `agents/process_guard/PROCESS_GUARD_AGENT.md` | P-01-P-18, P-21 (P-19/P-20 folgen nach Merge), Pre-Transition-Checks |
| **Review-Agent** | `agents/review_agent/REVIEW_AGENT_WORKFLOW.md` | R-01-R-30, Severity, Finding-Framework |

> **Beim Session-Start diese Dateien lesen**, bevor mit der Arbeit begonnen wird.

---

## Wer ist der Product Owner?

Der PO ist der GitHub-User mit **Schreibrechten auf** `SCHUNK-SE-Co-KG/TARATool`.
Der aktive Chat-Gespraechspartner ist der PO.

---

## WICHTIGSTE REGEL: Keine Arbeit ohne PO-Freigabe

**Keine Implementierung, kein Branch, keine Tests - ohne nachgewiesene Freigabe.**

Freigabe ist gegeben wenn eine der folgenden Bedingungen erfuellt ist:
1. Chat-Nachricht in der aktuellen Session enthaelt: `freigegeben`, `Freigabe fuer TARA-XXXX`, `PO-OK`, `akzeptiert`, `Accepted`, `Ok`
2. GitHub Issue-Kommentar am Epic oder Story enthaelt: `PO-OK`, `Freigabe erteilt`, `freigegeben`, `akzeptiert`, `Accepted`, `Ok`

---

## `/init` und Session-Start: Kein eigenstaendiger Arbeitsbeginn (Regel P-23)

> ⛔ **Diese Regel gilt VOR allen anderen Schritten** - auch und gerade wenn die
> Session mit dem eingebauten CLI-Befehl `/init` beginnt.

`/init` darf in diesem Repository **niemals** dazu fuehren, dass der Agent
eigenstaendig Dateien anlegt, ueberschreibt oder aendert (das schliesst
insbesondere ein automatisches Neuschreiben dieser Datei
`.github/copilot-instructions.md` durch die eingebaute `/init`-Routine der
CLI ein) - unabhaengig davon, was der Standard-`/init`-Ablauf der CLI sonst
vorsieht. `/init` in diesem Repo ist **ausschliesslich** ein Lese-/Analyse-
und Vorschlags-Vorgang:

1. **Lesen:** `docs/ENTWICKLUNGSPROZESS.md` vollstaendig parsen.
2. **Lesen:** alle `agents/*/*.md`-Dateien (Dev-Agent, Process-Guard,
   Review-Agent) vollstaendig parsen.
3. **Board sichten:** Projektboard laden (Blocking, In Progress, Todo-Epics/
   Stories, akzeptierte Review-Findings) gemaess Ablauf unten.
4. **Vorschlagen:** dem PO/User eine Zusammenfassung praesentieren, inkl.
   eines konkreten Vorschlags, mit welchen Issues/Epics als naechstes
   begonnen werden koennte - **ohne** diese Arbeit bereits zu beginnen.
5. **Warten:** Erst nach expliziter PO-/User-Freigabe (siehe "WICHTIGSTE
   REGEL" oben) duerfen Branch, Commit, Datei-Aenderungen oder
   Board-Status-Wechsel erfolgen.

Werden durch das Ausfuehren von `/init` (oder eine andere eingebaute
CLI-Routine) dennoch automatisch Datei-Aenderungen vorgenommen, **muessen**
diese sofort mit `git checkout --` bzw. `git restore` zurueckgesetzt werden,
bevor irgendeine weitere Aktion erfolgt.

---


## Schluesselwoerter in Issue-Kommentaren

Diese Liste ist deckungsgleich mit der tatsaechlichen Erkennung in
`scripts/process_guard/check_po_approval_keyword.sh` (aufgerufen von
`.github/workflows/po-approve.yml`, TARA-0086). Freigabe gilt sowohl bei
Kommentar im **Story-Issue** als auch im **Epic-Issue** (z.B. Sammelfreigabe
fuer alle Stories eines Epics).

| Schluesselwort | Bedeutung | Wirkung |
|----------------|-----------|---------|
| `PO-OK` | PO-Freigabe | Story/Epic freigegeben ODER Done-Setzen erlaubt |
| `Freigabe erteilt` | PO-Freigabe | Story/Epic freigegeben |
| `freigegeben` | PO-Freigabe | Story/Epic freigegeben |
| `akzeptiert` | PO-Freigabe | Story/Epic freigegeben |
| `Accepted` | PO-Freigabe | Story/Epic freigegeben |
| `Ok` / `OK` | PO-Freigabe | Story/Epic freigegeben |
| `Pause` | Arbeit pausiert | Issue wird im aktuellen Status belassen, nicht weiterbearbeitet |

> **Pause**: Ein Issue mit Kommentar `Pause` wird vom Dev-Agent in dieser Session **uebersprungen**.
> Es bleibt im aktuellen Status. Weiterarbeit nur nach erneutem expliziten PO-OK.

---

## Audit-Trail bei Board-Status-Wechseln (TARA-0086)

Bei **jedem** Statuswechsel (Todo -> In Progress -> inReview -> Freigabe -> Done)
hinterlaesst der Dev-Agent einen kurzen Kommentar im betroffenen Issue, z.B.:

> `P-02: Status Todo -> In Progress (PO-Freigabe: "akzeptiert", Kommentar von @po-user)`

Dies macht jeden Wechsel im Nachhinein nachvollziehbar (wann/warum/durch wen).


---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SCHUNK-SE-Co-KG/TARATool](https://github.com/SCHUNK-SE-Co-KG/TARATool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
