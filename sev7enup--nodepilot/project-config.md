---
trigger: always_on
description: Moderner, schlanker Ersatz fuer Microsoft System Center Orchestrator. Agentless Workflow-Orchestrierung fuer Windows-Umgebungen via WinRM.
---

# NodePilot

Moderner, schlanker Ersatz fuer Microsoft System Center Orchestrator. Agentless Workflow-Orchestrierung fuer Windows-Umgebungen via WinRM.

**Diese Datei ist der Index, nicht die Doku.** Sie enthält Verhaltensregeln, Nachschlage-Tabellen und Invarianten — die Tiefe liegt in `docs/`.

**Pflege:** Ein Feature-PR fügt hier höchstens eine Zeile hinzu. Wer einen Absatz schreiben will, schreibt ihn in `docs/claude-reference.md` und verlinkt ihn von hier. Keine Fehler-Rückblenden („vorher war es so…"), keine Messwerte als Beleg — dieselbe Regel wie für Code-Kommentare. Steht eine Erklärung schon in `docs/`, steht hier nur der Zeiger. Umfang ist per `DocumentationCountsTests` gedeckelt.

## Contributor- & Attribution-Policy

NodePilot ist ein Single-Contributor-Projekt. KI darf beim Entwickeln helfen (diese Datei, `.claude/`, `.agents/` bleiben in Nutzung) — aber für ALLE Commits ab v1.0.0 gilt:

- **Autor & Committer sind immer** `Sev7eNup <79143581+Sev7eNup@users.noreply.github.com>`. Der GitHub-Contributor-Graph zeigt ausschließlich `sev7enup`.
- **Co-Author-Trailer sind erlaubt** (Claude, Codex, Dependabot). Sie stehen immer *hinter* dem Autor: `Sev7eNup` bleibt Hauptautor und damit das erste Avatar am Commit. Ein Trailer darf den Autor nie ersetzen.
- **Natürliche, menschliche Sprache.** Alle Texte und Beschreibungen — ebenso PR-Titel, PR-Beschreibungen und Commit-Nachrichten — sind natürlich und menschlich formuliert. KI-Floskeln und aufgeblähte Formulierungen sind zu vermeiden, Aussagen bleiben konkret und verständlich.
- **Sprache auf GitHub ist Englisch.** Commit-Messages, PR-Titel/-Beschreibungen, Issues, Issue-Kommentare, Review-Kommentare und Branch-Namen werden auf Englisch verfasst — unabhängig davon, in welcher Sprache der Chat geführt wird. (Repo-interne Doku und Code-Kommentare bleiben davon unberührt: dort gilt weiter die vorhandene Sprache der jeweiligen Datei.)

## Agent skills

- **Issue tracker:** GitHub Issues für `Sev7eNup/NodePilot`, siehe `docs/agents/issue-tracker.md`
- **Triage labels:** Fünf-Label-Vokabular, siehe `docs/agents/triage-labels.md`
- **Domain docs:** Single-context repo → root `CONTEXT.md` + `docs/adr/`, siehe `docs/agents/domain.md`

## Doku-Landkarte

- `docs/roadmap.md` — **führendes Dokument für „was wird gebaut".** Gesetzte Posten (R1), trigger-gated Posten (R2), offene Entscheidungen (E), Sperrvermerk-Anhang mit den verworfenen Ideen. Was dort nicht steht, ist kein Vorhaben.
- `docs/claude-reference.md` — **Overflow-Referenz dieser Datei.** Activity-Config-Keys/Outputs, Trigger-Params, Edit-Lock-UX, Audit-Codes, Hot-Reload-Matrix, Backup-Details, Background-Services, Deployment, Coverage-Messung
- `docs/alerting.md` — Notification-Rules + System-Policies (ADR 0008), Dispatcher, Sinks, Ledger
- `docs/custom-activities.md` — Custom Activities (Plugin-System)
- `docs/mcp-server.md` — MCP-Server inkl. Tool-Katalog + `.mcp.json`-Beispiel
- `docs/ai-features.md` — KI-Features: Config-Keys, Modell-Empfehlungen
- `docs/deployment-guide.md` — (EN) **nicht** der Installationsweg, sondern was davor und danach kommt: Artefakt verifizieren, selbst bauen, Troubleshooting. Der Installationsweg steht **einmal**, auf der Doku-Website (`content/{de,en}/deployment/production.md`).
- `docs/av-exclusions.md` — Antiviren-Ausschlüsse (Server + Desktop) als Übergabedokument für eine AV-Abteilung
- `docs/workflow-styleguide.md` — Layout-Styleguide für Workflow-JSONs (**vor jedem Workflow-Gen lesen**)
- `docs/workflow-tests.md` — Test-Suite unter `scripts/test-suite/`: 46 generierte Workflows gegen die laufende Engine, `suite-manifest.json` als Abdeckungsquelle, Guard-Test `TestSuiteCoverageTests`
- `docs/enterprise-features.md` — HA, Secret-Provider, LDAP/SSO, SIEM, Folder-RBAC
- `docs/ai-feature-ideas.md` — Beschreibungstiefe zu den KI-Ideen, **keine Spezifikation**. Priorisierung und Status stehen in `docs/roadmap.md`.
- `src/nodepilot-ui/e2e/README.md` — E2E-Coverage-Map + Spec-Konventionen
- `src/nodepilot-ui/demo/` — **Browser-Demo** der SPA für GitHub Pages (`/demo/`): dieselbe App gegen ein In-Memory-Backend. Eigener Entry, dritter Vite-Build (`vite.demo.config.ts` → `dist-demo/`). Abhängigkeitsrichtung ausschließlich `demo/` → `src/`; Hub, Doku-Link und Auth-Transport werden vom Demo-Entry **injiziert**. Details: `src/nodepilot-ui/CLAUDE.md`
- `src/nodepilot-docs-ui/src/site/` — **Projekt-Website** (DE/EN, Vanilla-TS) an der Pages-Wurzel, Doku darunter unter `/docs/`. Eigener Vite-Build, **nie** Teil von `dist/` oder des Produkts. `src/nodepilot-docs-ui/scripts/assemble-site.mjs` baut `_site/`; **jede Eingabe ist Pflicht** — eine optionale ließe einen Deploy die alte Demo weiterveröffentlichen. `npm run preview:site` ist die einzige vollständige lokale Vorschau. Routen sind **echte Adressen** (`/product/`, englische Segmente, Rechtsseiten deutsch), die das Prerender-Plugin nach dem Build je Route als eigene Datei schreibt, samt `404.html`, `sitemap.xml` und `robots.txt`. Neue Route → `ROUTE_PATHS`, `routePages()` **und** `SITE_ROUTE_SEGMENTS`. Impressum/Datenschutz liefert der User. Details: `src/nodepilot-docs-ui/README.md`

## Tech-Stack


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sev7eNup/NodePilot](https://github.com/Sev7eNup/NodePilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
