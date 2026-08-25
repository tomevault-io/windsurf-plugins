---
trigger: always_on
description: **Prozess-Stand: v1.9.1** — Stand der Vorlage, aus der dieses Projekt stammt.
---

# CLAUDE.md — Projektanweisungen

**Prozess-Stand: v1.9.1** — Stand der Vorlage, aus der dieses Projekt stammt.
Beim Abgleich mit einer neueren Vorlagen-Version hochsetzen; wie das geht, steht
in `docs/agents/abgleich.md` im Vorlagen-Repo `Trust1509/agent-projekt-template`
(dieses Repo führt selbst keine `abgleich.md`, weil sie nur beim Abgleichen
gebraucht wird, nicht im laufenden Betrieb). Diese Zeile bleibt im Projekt
stehen.

Immich Family Tools ist eine Companion-Web-App für selbst gehostetes Immich:
ein FastAPI-Backend und ein React/TS-Frontend in einem gemeinsamen Container,
für kontenübergreifendes Gesichts-Matching und geteilte Alben über die
Immich-REST-API. Es gibt keine eigene Datenbank — die gesamte Persistenz ist
eine einzige JSON-Datei (`accounts.json`) auf einem ZFS-Volume. Das Projekt
**erfordert Immich v3.x**.

**Profil: Anwendung ohne Datenbank.** Die gesamte Persistenz ist eine JSON-Datei;
es gibt kein Migrations-Framework, keinen Datenbank-Dienst und keine öffentlich
erreichbare Schnittstelle. Der **Prozess-Kern** (Panel, Bau-Brief, Rot-Beweis,
Release-Ritual, Betrieb) gilt vollständig. Die **Stack-Maschine**
(Migrationsköpfe, Roundtrips gegen eine echte Zieldatenbank,
Schnittstellen-Fuzzing) hat hier keinen Gegenstand — mit einer Ausnahme, die
zählt: Die Datenerhalt-Probe gilt sehr wohl, nur framework-frei, als Test gegen
das JSON-Alt-Format (`backend/tests/test_config_store.py`).

## Vor dem ersten Slice

**`docs/agents/lehren.md` lesen.** Fehlerklassen, die real getroffen haben —
Wächter, die grün sind ohne etwas zu beweisen; Seitenkanäle; Datenmigrationen,
deren Fehler kein Update mehr heilt; „weniger Abfragen", das langsamer ist —
plus ein eigener Abschnitt mit Funden aus diesem Projekt. Kostet fünf Minuten
und hat schon mehrfach einen Prod-Fund verhindert.

Für die meisten Schritte gibt es fertige Methoden-Anleitungen — welche, woher
und wann welche passt: `docs/agents/skills.md`.

## Arbeitsweise

**Issues sind der Arbeitsspeicher.** Jeder Fund, jede Entscheidung, jeder
zurückgestellte Punkt wird ein Issue — nicht eine Notiz im Chat. Siehe
`docs/agents/issue-tracker.md` und `docs/agents/triage-labels.md`.

**Ein Slice nach dem anderen.** Parallel nur, was sich nachweislich nicht
überschneidet (verschiedene Dateien, verschiedene Schichten). Sonst arbeiten zwei
Bauläufe im selben Arbeitsbaum und niemand weiß mehr, wem welche Änderung gehört.

**Der Hauptagent baut nicht selbst, er arbitriert.** Bauen macht ein Subagent mit
einem Bau-Brief (`docs/agents/bau-brief.md`), prüfen macht das Panel
(`docs/agents/panel.md`), entscheiden macht der Hauptagent — **und reproduziert
jeden Blocker am Code**, statt der Konvergenz der Prüfer zu glauben.

**Technisches selbst entscheiden, Fachliches fragen.** Bibliothekswahl,
Schnittstellen-Zuschnitt, Testform: selbst. Ob eine Zahlung auf eine
abgeschlossene Rechnung möglich sein soll: fragen. Im Zweifel: eine Annahme
formulieren, weiterbauen, die Annahme sichtbar machen.

## Review-Panel

Verbindlich nach **jedem nicht-trivialen Slice**, vor dem Landen. **Trivial ist
abschließend nur:** reine Testinfrastruktur ohne Verhaltensänderung (ein neuer
Test zählt NICHT dazu — er behauptet etwas über Verhalten), Doku-Korrektur ohne
ausgelieferten Inhalt, Typisierung ohne Verhaltensänderung. **Immer volles
Panel** bei Datenmigrationen, Datenschutz-/Berechtigungslogik, allem mit
Geld-/Steuerbezug, allem über eine Schnittstelle nach außen — und allem mit
**Herkunft**: Code, den niemand aus dem Team gebaut hat (Fremdmodell,
zugelieferter Zweig, übernommenes Beispiel).

Drei Stimmen (blinde Erststimme, unabhängiges Modell, günstige Drittstimme),
Ablauf, Arbitrierung und Panel-Kommentar-Form stehen vollständig in
`docs/agents/panel.md` — keine Kurzfassung darüber hinaus, um Doppelpflege zu
vermeiden. Modellnamen und Aufrufkommandos der Stimmen 2 und 3 für dieses
Projekt stehen ebenfalls dort.

## Bau-Brief

Jeder Bau-Auftrag folgt dem Pflicht-Gerüst aus `docs/agents/bau-brief.md` —
acht Blöcke, keiner leer. **Vor jedem Absenden verbindlich:**
`sh scripts/bau-brief-pruefen.sh <brief.md>`. Details, Begründungen und
projektspezifische Prüf-Kommandos stehen dort, nicht hier.

## Release

**Schwelle „gefahrlos":** nur Code, keine Migration — dafür gilt bei grüner CI
eine stehende Owner-Freigabe zum Taggen; jede Migration (Stufe backup oder
breaking) bleibt Owner-Sache und wird vor dem Tag gefragt, im Zweifel die
vorsichtigere Stufe. Ablauf, welche Dateien die Version führen und alle
Details stehen in `docs/agents/release-ritual.md` — keine weitere
Kurzfassung hier.

## Prüfschritte

Ein Prüfschritt, den nur die CI kennt, wird lokal nie gefahren und meldet sich
zum ungünstigsten Zeitpunkt — genau so ist `npm audit` in diesem Repo einmal
rot geworden, mitten in einem Release, ohne dass sich eine Zeile geändert
hatte. Deshalb hier **alle** Kommandos, die die CI fährt:

**Push-Pfad (`.github/workflows/ci.yml`, bei jedem Push auf `main`/`release/**`
und bei Tags):\*\*

- Backend: `pytest -q backend/tests`
- Backend: `python -m compileall -q backend`
- Frontend (in `frontend/`): `npm ci`
- Frontend (in `frontend/`): `npm test`
- Frontend (in `frontend/`): `npm run build` (enthält `tsc`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Trust1509/immich-family-tools](https://github.com/Trust1509/immich-family-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
