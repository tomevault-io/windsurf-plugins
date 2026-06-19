---
trigger: always_on
description: >
---


# Skill Forge

Iterative Verbesserung nach dem Autoresearch-Paradigma: Ein AI-Agent modifiziert
gezielt Dateien, evaluiert jede Änderung gegen eine mechanische Metrik, behält
Verbesserungen und verwirft Verschlechterungen.

## Ausführungsmodi

| | Auto-Modus | Guided-Modus |
|---|---|---|
| **Ablauf** | Vollautonomer Loop ohne User-Eingriff | User entscheidet an jedem Checkpoint |
| **Ideal für** | Overnight-Runs, Scheduled Tasks | Erstmalige Nutzung, Domänenwissen einbringen |
| **Evals** | Automatisch generiert | User prüft und passt an |
| **Hypothesen** | Automatisch umgesetzt | User sieht Vorschlag, kann ablehnen/anpassen |
| **Mutationen** | Automatisch angewendet | User sieht Diff, bestätigt oder korrigiert |
| **Keep/Revert** | Automatisch nach Schwellenwerten | User entscheidet mit Score als Empfehlung |
| **Wann wählen** | Vertrautes Setup, bewährte Evals | Neuer Skill, unsichere Evals, Lernmodus |

Der Wizard fragt als ersten Schritt: **"Auto oder Guided?"**

Im Guided-Modus gibt es 5 Checkpoints, an denen der User einbezogen wird:

1. **Evals prüfen** — User sieht generierte Evals, kann anpassen/ergänzen/streichen, Anzahl und Gewichtung bestimmen
2. **Hypothese prüfen** — User sieht die Hypothese und kann sie ablehnen, anpassen oder eine eigene Richtung vorgeben
3. **Mutation prüfen** — User sieht das Diff vor Anwendung und bestätigt
4. **Ergebnis bewerten** — User sieht Score + Delta und entscheidet: Keep, Revert oder manuell anpassen
5. **Weitermachen?** — User entscheidet ob eine weitere Runde laufen soll oder der Loop endet

Im Auto-Modus werden alle 5 Checkpoints übersprungen und die Entscheidungen
automatisch nach den konfigurierten Schwellenwerten getroffen.

## Zwei Domänen-Modi

| | Skill-Modus | Generic-Modus |
|---|---|---|
| **Ziel** | SKILL.md verbessern | Beliebige Dateien optimieren |
| **Metrik** | Composite Score (Assertions + Judge + Effizienz) | Jede mechanische Metrik (Zahl via Shell-Command) |
| **Scope** | Eine SKILL.md + zugehörige Scripts | Dateien via Glob-Pattern |
| **Mutation** | Sprachliche Änderungen (Formulierung, Beispiele, Struktur) | Code-Änderungen (Refactoring, Config, Architektur) |
| **Eval** | Subagent-Runs mit Grading | Shell-Command mit Zahlenextraktion |
| **Anwendung** | Skill-Qualität steigern | Testcoverage, Bundle-Size, Lighthouse, Docker-Image, etc. |

Der Skill erkennt den Modus automatisch: Wenn der User einen Skill nennt, wird
Skill-Modus aktiviert. Wenn der User eine Metrik/einen Shell-Command nennt, wird
Generic-Modus aktiviert. Im Zweifel: fragen.

## Kern-Konzept

Inspiriert von Karpathys autoresearch-Paradigma:

| autoresearch (LLM)        | autoresearch (Skills/Generic)  |
|---------------------------|-------------------------------|
| `train.py` wird mutiert   | `SKILL.md` / Scope-Dateien werden mutiert |
| `prepare.py` ist fix      | Eval-Framework / Verify-Command ist fix |
| `program.md` instruiert   | Dieser Skill instruiert       |
| `val_bpb` ist die Metrik  | Composite Score / mechanische Metrik |
| 5-Min-Zeitbudget          | Token/Zeit-Budget pro Eval    |
| keep/discard              | keep/revert mit Snapshots     |

---

## Schritt 0: Setup-Wizard (einmalig)

Der Setup-Wizard führt schrittweise durch die Konfiguration. Jeder Schritt hat ein
Abnahmekriterium — der Wizard geht erst weiter, wenn die Validierung bestanden ist.

### Wizard-Schritt 1: Ausführungsmodus, Domänenmodus und Ziel erfassen

Frage den User zwei Dinge:

**1. "Auto oder Guided?"**
- **Auto**: "Ich lasse den Loop laufen und schaue mir morgens den Report an"
- **Guided**: "Ich will bei jedem Schritt mitentscheiden"
- Bei Scheduled Tasks: Immer Auto (Guided nicht möglich ohne User)

**2. "Was willst du verbessern?"**

Bestimme daraus den Domänenmodus:
- User nennt einen Skill-Namen → **Skill-Modus**
- User nennt eine Metrik, einen Shell-Command oder Code-Dateien → **Generic-Modus**
- Unklar → Nachfragen

Speichere:
```json
{
  "execution_mode": "auto" | "guided",
  "mode": "skill" | "generic",
  "goal": "Freitext-Beschreibung des Ziels",
  "target": "Skill-Name oder Projekt-Pfad"
}
```

### Wizard-Schritt 2: Scope definieren

**Skill-Modus:**
- Identifiziere die SKILL.md des Target-Skills
- Validierung: Datei existiert und ist lesbar

**Generic-Modus:**
- Frage nach Glob-Pattern für editierbare Dateien (z.B. `src/**/*.ts`)
- Validierung: Glob matcht mindestens eine Datei
- Anzeigen: "Gefunden: N Dateien — [Liste der ersten 10]"
- Falls 0 Treffer → Fehlermeldung, neues Pattern verlangen

Speichere:
```json
{
  "scope": "SKILL.md-Pfad oder Glob-Pattern",
  "scope_files_count": 42,
  "scope_validated": true
}
```

### Wizard-Schritt 3: Metrik definieren

**Skill-Modus:**
- Prüfe ob `evals/evals.json` existiert im Target-Skill
- Falls nicht: Erstelle 3-5 realistische Testfälle mit messbaren Assertions
- Achte auf Train/Test-Split (60/40) für Overfitting-Schutz
- Metrik = Composite Score (automatisch)

**🔀 Guided-Checkpoint 1: Evals prüfen (nur Skill-Modus)**

Im Guided-Modus: Zeige dem User die generierten/vorhandenen Evals und frage:
- "Das sind die Testfälle. Passen sie?"
- User kann: Evals anpassen, neue hinzufügen, Gewichtung ändern, Anzahl bestimmen
- User kann auch beschreiben: "Ich will dass besonders X getestet wird"
- Erst nach User-Bestätigung wird der Train/Test-Split durchgeführt

**Generic-Modus:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GodModeAI2025/skill-forge](https://github.com/GodModeAI2025/skill-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
