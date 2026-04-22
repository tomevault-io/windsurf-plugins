---
trigger: always_on
description: Repo-Anweisungen fuer Codex und andere Coding Agents bei der Arbeit an MasterSelects.
---

# AGENTS.md

Repo-Anweisungen fuer Codex und andere Coding Agents bei der Arbeit an MasterSelects.

Diese Datei ist das Codex-Pendant zu `CLAUDE.md`. Wenn sich Workflow, Branch-Regeln, Debug-Rezepte oder Projektkonventionen aendern, `AGENTS.md` und `CLAUDE.md` gemeinsam pflegen.

---

## 0. Projektziel / Vision

MasterSelects soll bis Juni 2026 ALLE Media-Dateien unterstuetzen, nicht nur Video, Audio und Bilder, sondern wirklich alles:

- 3D: OBJ, FBX, glTF
- Dokumente: PDF, SVG
- CAD / technische Daten: DXF, STEP
- Datenformate: JSON, CSV, binaere Formate, Point Clouds und mehr

Inspiration ist das TouchDesigner-Prinzip: Jede Datei wird zu einem visuellen Signal. Es gibt keine "nicht unterstuetzten" Dateien. Alles wird Textur, Geometrie oder Daten, kann auf der Timeline platziert, composited und exportiert werden.

## 0.1 Codex Skill Mapping

Die alten Claude-Commands unter `C:\Users\admin\.claude\commands\` sollen in Codex ueber Skills abgebildet werden. Wenn die folgenden Skills verfuegbar sind, diese bevorzugen:

| Claude Command | Codex Skill | Zweck |
|---|---|---|
| `/masterselects` | `masterselects` | Timeline-, Clip-, Preview- und Analyse-Aktionen ueber die lokale AI-Bridge |
| `/cloudflare` | `cloudflare-api` | Cloudflare REST / Wrangler |
| `/stripe` | `stripe-api` | Stripe REST API |
| `/vazer` | `vazer-app-api` | Lokale VAZer App / XML / Analyse |
| `/react-doctor` | `react-doctor` | React-Codebase-Analyse |
| `/nano-banana` | `nano-banana` | Bildgenerierung via Gemini |
| `/kie` | `kie-ai-api-route` | Kie.ai Bild- und Video-Generation |
| `/kling` | `kling` | Kling Video Prompting / API |
| `/tasks` | `tasks` | Aufgabenliste |
| `/email` | `email` | Strato / OX Mail |
| `/gmail` | `gmail` | Gmail per IMAP/SMTP |
| `/dienstplan` | `dienstplan` | PDF-Dienstplan -> Kalender |

Wenn ein Skill nicht verfuegbar ist, direkt ueber lokale Skripte, MCP-Tools, HTTP-Bridge oder API arbeiten. Nicht an Claude-Command-Dateien haengen bleiben.

## 0.2 gstack Integration

`gstack` ist fuer strukturierte Planung, Reviews, Browser-QA und Security-Checks verfuegbar. Global installieren, nicht ins Repo vendorisieren.

Installationspfade:

- Codex: `git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/gstack && cd ~/gstack && ./setup --host codex`
- Claude Code: `git clone --single-branch --depth 1 https://github.com/garrytan/gstack.git ~/.claude/skills/gstack && cd ~/.claude/skills/gstack && ./setup --team`
- Windows: Git Bash oder WSL verwenden; `bun` und `node` muessen installiert sein
- Nach der Codex-Installation Codex neu starten, damit neue Skills geladen werden

Einsatzregeln:

- `masterselects` bleibt erste Wahl fuer Timeline-, Preview-, Clip- und Debug-Bridge-Automation in der lokalen App
- `gstack-office-hours`, `gstack-autoplan`, `gstack-plan-eng-review`, `gstack-plan-design-review` und `gstack-plan-devex-review` fuer Discovery, Scope und Plan-Qualitaet
- `gstack-review` fuer unabhaengige Code-Reviews
- `gstack-investigate` fuer Root-Cause-Debugging statt Trial-and-Error-Fixes
- `gstack-cso` fuer Security-Reviews
- `gstack-qa`, `gstack-qa-only`, `gstack-browse`, `gstack-open-gstack-browser` und `gstack-setup-browser-cookies` fuer Browser-QA, Repros und auth-geschuetzte Flows
- `gstack-upgrade` verwenden, statt eine Repo-lokale gstack-Kopie zu pflegen

## 0.3 MasterSelects Debug Bridge

Fuer App-Debugging existieren lokale AI-Tools hinter `POST http://localhost:5173/api/ai-tools`. Voraussetzung: Dev-Server laeuft und die App ist im Browser geoeffnet.

| Tool | Parameter | Zweck |
|---|---|---|
| `getStats` | none | Engine-Snapshot: FPS, Timing, Decoder, Drops, Audio, GPU |
| `getStatsHistory` | `samples?`, `intervalMs?` | Mehrere Snapshots mit min/max/avg |
| `getLogs` | `limit?`, `level?`, `module?`, `search?` | Browser-Logs gefiltert abrufen |
| `getPlaybackTrace` | `windowMs?`, `limit?` | WebCodecs- und VF-Pipeline-Events plus Health-State |

Der `masterselects`-Skill ist der bevorzugte Einstieg. Wenn der Skill nicht nutzbar ist, die HTTP-Bridge direkt per `curl` ansprechen.

---

## 1. Workflow

### Branch-Regeln

| Branch | Zweck |
|---|---|
| `staging` | Entwicklung, Standardziel fuer laufende Arbeit |
| `master` | Production, nur via PR |

### Commit- und Push-Regeln

Vor jedem Commit alle Checks ausfuehren:

```bash
npm run build
npm run lint
npm run test
```

Regeln:

- Nie direkt auf `master` committen.
- Nie selbststaendig nach `master` mergen.
- Nie selbststaendig pushen, ausser der User verlangt es explizit.
- Kleine, zusammenhaengende Aenderungen bevorzugen.
- Nicht committen, wenn Build, Lint oder Tests fehlschlagen.

### Merge zu Master

Nur wenn der User es explizit verlangt:

1. Version in `src/version.ts` erhoehen.
2. CHANGELOG in `src/version.ts` aktualisieren.
3. Commit und Push.
4. PR von `staging` nach `master` erstellen und mergen.
5. `staging` wieder auf den aktuellen `master`-Stand bringen.

### Version / Changelog

- Datei: `src/version.ts`
- Version nur bei Merge nach `master` erhoehen
- CHANGELOG immer am Anfang erweitern
- `KNOWN_ISSUES` aktuell halten

### Dokumentation

Bei Feature-Aenderungen relevante Doku in `docs/Features/` aktualisieren.

---

## 2. Quick Reference

```bash
npm install && npm run dev

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sportinger/MasterSelects](https://github.com/Sportinger/MasterSelects) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
