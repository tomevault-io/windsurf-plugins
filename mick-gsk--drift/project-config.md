---
trigger: always_on
description: **Diese Datei ist für alle Copilot-Agenten, Coding-Agenten und KI-Assistenten im Drift-Workspace bindend.**
---

# Drift — Verbindliche Arbeitsgrundlage für alle Agenten

**Diese Datei ist für alle Copilot-Agenten, Coding-Agenten und KI-Assistenten im Drift-Workspace bindend.**

Die vollständige Policy befindet sich in:
`POLICY.md` (Workspace-Root)

Lies diese Datei **vor jeder Arbeit** vollständig, sofern sie nicht bereits im Kontext ist.
Die Policy ist ein Vertrag — keine Empfehlung, kein Vorschlag.

---

## Primärmodus bei Prompt-Engineering

Wenn eine Aufgabe Prompts, Instructions, Skills, Agents oder diese Datei selbst betrifft,
arbeitet der Agent im **Prompt-Engineering-Modus**. Ziel ist nicht schönere Prosa,
sondern härtere, operativere und reviewbare Agentensteuerung.

**Betroffene Pfade:**
- `.github/prompts/**`
- `.github/instructions/**`
- `.github/skills/**`
- `.github/agents/**`
- `.github/AGENTS.md`
- `.github/copilot-instructions.md`

**Detaillierte Zusatzregeln:**
- Routing: `.github/instructions/drift-context-routing.instructions.md`
- Prompt-Engineering: `.github/instructions/drift-prompt-engineering.instructions.md`
- Workflow-Skill: `.github/skills/drift-agent-prompt-authoring/SKILL.md`

### Zuerst das richtige Primitive wählen

| Bedarf | Richtiges Primitive |
|--------|---------------------|
| Repo-weite, immer geltende Regeln | `copilot-instructions.md` |
| Datei- oder Ordner-spezifische Regeln | `*.instructions.md` |
| Wiederverwendbarer Operator-Workflow | `SKILL.md` |
| Konkreter mehrphasiger Ablauf mit Ziel und Artefakten | `*.prompt.md` |
| Isolierter Spezialmodus mit eigener Tool-/Kontextgrenze | `*.agent.md` |

**Falsches Primitive = schwacher Prompt.**
Eine Datei darf nur das regeln, wofür ihr Primitive gedacht ist.

### Nicht verhandelbare Schärferegeln

1. **Deutsch und modellunabhängig.** Keine Modellnamen, keine vendor-spezifischen Prompt-Tricks.
2. **Description ist Discovery-Oberfläche.** Das `description`-Feld muss Triggerwörter,
   Task-Typ und Scope explizit enthalten.
3. **`applyTo` so eng wie möglich.** `applyTo: "**"` nur für wirklich universelle Regeln.
4. **Ein Problem pro Datei.** Keine Mischdateien für Policy, Testing, Release und Prompt-Stil.
5. **Keine Parallel-Policy.** Shared Partials, Policy und Push-Gates referenzieren statt duplizieren.
6. **Keine weichen Verben ohne Vertrag.** Wörter wie "analysiere", "verbessere",
   "prüfe gründlich", "wenn möglich" oder "robust" sind nur zulässig, wenn Inputs,
   Schritte, Ergebnisform und Abbruchkriterium definiert sind.
7. **Keine Halluzinationsflächen.** Keine erfundenen Flags, Dateien, Tools, Issue-Ziele,
   Pfade oder angeblich vorhandenen Artefakte.
8. **Prompts erzeugen Evidenz.** Jeder Prompt muss in Beobachtung, Artefakt, Entscheidung,
   Maßnahme oder sauberem Abbruch enden - nie nur in Prosa.

### Mindestvertrag für jeden guten Prompt

Jeder neue oder geänderte Prompt, Skill oder jede Instruction muss explizit benennen:

- Ziel und Scope
- Eingaben, Voraussetzungen und relevante Umgebungsannahmen
- welches Tool oder welcher Befehl wofür eingesetzt wird
- erwartete Artefakte oder das Ausgabeformat
- Bewertungslogik oder Entscheidungskriterien
- Fehlerpfad, Fallback oder Eskalation
- klare Stop-Bedingung
- referenzierte Single Sources of Truth

Fehlt einer dieser Punkte, ist die Anweisung zu weich.

### Single Sources of Truth für Prompt-Arbeit

Diese Dateien werden wiederverwendet statt kopiert:

- `.github/prompts/_partials/konventionen.md`
- `.github/prompts/_partials/bewertungs-taxonomie.md`
- `.github/prompts/_partials/issue-filing.md`
- `.github/prompts/_partials/issue-filing-external.md`
- `.github/skills/drift-agent-prompt-authoring/SKILL.md`
- `.github/instructions/drift-policy.instructions.md`
- `.github/instructions/drift-context-routing.instructions.md`
- `.github/instructions/drift-prompt-engineering.instructions.md`

### Repo-spezifische Prompt-Regeln

- Interne Drift-Prompts arbeiten gegen den Workspace und referenzieren die Dev-Version.
- Field-Test-Prompts arbeiten gegen externe Repositories und reichen Issues immer an
  `mick-gsk/drift`, nie an das Ziel-Repository.
- Prompt-Arbeit ist nur zulässig, wenn sie Erkenntnis, Vergleichbarkeit, Einführbarkeit
  oder Signalqualität verbessert - nicht wenn sie nur mehr Text produziert.

---

## Policy als Single Source of Truth

`POLICY.md` ist die alleinige Quelle fuer Produkt- und Priorisierungsregeln. Diese Datei dupliziert **nicht** mehr die Inhalte aus Policy §6, §8, §13, §14, §16 oder §18.

Fuer Agentenarbeit gilt deshalb:

- Policy und Gate-Logik: `POLICY.md` und `.github/instructions/drift-policy.instructions.md`
- Risk-Audit-Pflichten: `.github/instructions/drift-policy.instructions.md`
- Push-Vorbereitung: `.github/instructions/drift-push-gates.instructions.md`
- Release-Automation: `.github/instructions/drift-release-automation.instructions.md` und `.github/instructions/drift-release-mandatory.instructions.md`
- MCP-Fix-Loop: `.github/prompts/drift-fix-loop.prompt.md`

Wenn diese Datei und eine Single Source of Truth kollidieren, gilt immer die Single Source of Truth.

---

## Drift-Version-Freshness (Pflicht für alle Agenten)

Jeder Agent, der `drift` ausfuehrt, analysiert oder konfiguriert, MUSS sicherstellen,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mick-gsk/drift](https://github.com/mick-gsk/drift) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
