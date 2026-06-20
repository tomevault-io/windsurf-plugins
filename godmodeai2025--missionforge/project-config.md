---
trigger: always_on
description: >
---


# Mission Forge — Agent-Company-Spawner & Orchestrierungs-Engine

> Verwandelt jede Aufgabe in eine lebende Organisation aus spezialisierten Agenten
> mit garantierter Vollstaendigkeit, Nachverfolgbarkeit und Qualitaetssicherung.
> Revisionssicher durch kryptographische Hash-Chain. Exportiert bewährte Companies
> als wiederverwendbare Packages.

---

## Inhaltsverzeichnis

1. [Wann verwenden](#1-wann-verwenden)
2. [Kernkonzepte](#2-kernkonzepte)
3. [Phase 1 — Aufgabenanalyse & Zerlegung](#3-phase-1--aufgabenanalyse--zerlegung)
4. [Phase 2 — Company spawnen](#4-phase-2--company-spawnen)
5. [Phase 3 — Orchestrator-Hierarchie](#5-phase-3--orchestrator-hierarchie)
6. [Phase 4 — Skill-Zuordnung](#6-phase-4--skill-zuordnung)
7. [Phase 5 — Wellenplanung](#7-phase-5--wellenplanung)
8. [Phase 6 — Ausfuehrung](#8-phase-6--ausfuehrung)
9. [Phase 7 — Verifikation](#9-phase-7--verifikation)
10. [Phase 8 — Abschluss](#10-phase-8--abschluss)
11. [Phase 9 — Export & Wiederverwendung](#11-phase-9--export--wiederverwendung)
12. [Referenzen](#12-referenzen)

---

## 1. Wann verwenden

Aktiviere Mission Forge wenn:

- Eine **komplexe Aufgabe** mehrere Arbeitsschritte, Skills oder Perspektiven erfordert
- **Mehrere Skills** orchestriert werden muessen ohne dass etwas vergessen wird
- Eine **Ablauforganisation** mit klaren Verantwortlichkeiten gebraucht wird
- Eine **wiederverwendbare Organisationsstruktur** fuer wiederkehrende Aufgabentypen entstehen soll
- **Revisionssicherheit** gefordert ist (regulierte Branchen, Audits, Compliance)

**Nicht verwenden** fuer triviale Einzel-Tasks die in unter 2 Minuten erledigt sind.

---

## 2. Kernkonzepte

### 2.1 Die 6 Manifest-Typen (Agent Companies Standard)

| Manifest     | Zweck                                      | Datei        |
|--------------|--------------------------------------------|--------------|
| **COMPANY**  | Wurzel der Organisation, Ziele, Governance | `COMPANY.md` |
| **TEAM**     | Wiederverwendbarer Organisationsbaum       | `TEAM.md`    |
| **AGENT**    | Einzelne Rolle mit Anweisungen und Skills  | `AGENTS.md`  |
| **PROJECT**  | Geplante Arbeitsgruppierung                | `PROJECT.md` |
| **TASK**     | Atomare ausfuehrbare Arbeitseinheit        | `TASK.md`    |
| **SKILL**    | Wiederverwendbare Faehigkeit               | `SKILL.md`   |

Vollstaendige Feld-Referenz: [references/manifest-reference.md](references/manifest-reference.md)

### 2.2 Progressive Disclosure (3 Stufen)

1. **Katalog** — Nur Name + Beschreibung laden (~100 Token pro Einheit)
2. **Aktivierung** — Vollstaendige Manifeste bei Bedarf laden
3. **Ressourcen** — Scripts, Referenzen, Assets nur bei Ausfuehrung

### 2.3 Status-Werte (einheitlich in allen Dateien)

| Status        | Bedeutung                                          |
|---------------|-----------------------------------------------------|
| `OPEN`        | Noch nicht begonnen                                 |
| `IN_PROGRESS` | Agent arbeitet daran                                |
| `DONE`        | Agent hat Ergebnis geliefert                        |
| `VERIFIED`    | Ergebnis gegen Akzeptanzkriterien geprueft, bestanden|
| `FAILED`      | Reparatur-Versuche erschoepft                       |
| `ESCALATED`   | An hoehere Ebene eskaliert                          |
| `SKIPPED`     | Bewusst ausgelassen (mit Begruendung)               |
| `ABORTED`     | Mission vorzeitig beendet                           |

### 2.4 Prioritaeten (einheitlich)

| Prioritaet | Verwendung                          |
|------------|--------------------------------------|
| `critical` | Blocker, ohne dies geht nichts weiter|
| `high`     | Kernfunktionalitaet                  |
| `medium`   | Wichtig aber nicht blockierend       |
| `low`      | Nice-to-have                         |

### 2.5 Vollstaendigkeitsgarantie (Zero-Drop-Prinzip)

Jede Anforderung muss:
1. In mindestens einer TASK.md erfasst sein (Traceability)
2. Einem Agenten zugewiesen sein (Ownership)
3. In einer Welle eingeplant sein (Scheduling)
4. Nach Ausfuehrung verifiziert sein (Verification)
5. Im Abschlussbericht dokumentiert sein (Documentation)
6. In der AuditChain protokolliert sein (Revisionssicherheit)

Fehlt ein Schritt, blockiert die Pipeline.

### 2.6 AuditChain (Kryptographische Beweiskette)

Jede Zustandsaenderung in einer Mission wird als Hash-Chain-Eintrag protokolliert. Jeder Eintrag referenziert den SHA-256-Hash des vorherigen — Manipulation eines Eintrags bricht die Kette und wird bei der Verifikation sofort erkannt.

**Chain-Datei:** `.mission-forge/audit/CHAIN.jsonl`  
**Engine:** `audit/chain.py` (Python 3.10+, keine externen Dependencies)  
**Verifier:** `audit/verify.py` (Standalone, fuer externe Auditoren)

**Pflicht-Events:**

| Event                  | Wann                              | Phase |
|------------------------|-----------------------------------|-------|
| `GENESIS`              | Company wird gespawnt (inkl. Skill-Hashes) | 2     |
| `SKILL_CHANGED`        | Skill wurde mutiert/aktualisiert   | 6     |
| `GATE_PASSED`          | Aktion durch Gateway freigegeben   | 6     |
| `GATE_BLOCKED`         | Aktion durch Gateway blockiert     | 6     |
| `WAVE_PLAN_SEALED`     | Wellenplan freigegeben             | 5     |
| `TASK_STATUS_CHANGE`   | Jeder Statuswechsel eines WP      | 6     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GodModeAI2025/MissionForge](https://github.com/GodModeAI2025/MissionForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
