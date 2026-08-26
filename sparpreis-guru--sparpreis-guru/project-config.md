---
trigger: always_on
description: Diese Datei beschreibt den verlässlichen Arbeits- und Prüfablauf für dieses Repository.
---

# Codex-Projektleitfaden

Diese Datei beschreibt den verlässlichen Arbeits- und Prüfablauf für dieses Repository.

## Paketmanager

- Die im Feld `packageManager` der `package.json` festgelegte pnpm-Version ist verbindlich und wird über Corepack verwendet.
- Unter Windows nicht `pnpm` oder `pnpm.cmd` direkt aufrufen. Der globale Wrapper unter `%LOCALAPPDATA%\pnpm\bin` kann in dieser Umgebung hängen.
- Stattdessen immer `corepack pnpm <befehl>` verwenden.
- Beispiele:
  - Entwicklung: `corepack pnpm dev`
  - Abhängigkeiten installieren: `corepack pnpm install`
  - Produktions-Build: `corepack pnpm build`

## Verifikation

Die Prüfung soll proportional zur Änderung erfolgen. Solange aktiv an einem Feature gearbeitet wird, müssen nach Zwischenständen nicht automatisch Typprüfung, Git-Prüfung oder Browser-Test ausgeführt werden. Erst wenn das Feature erkennbar fertig entwickelt ist oder der Nutzer ausdrücklich eine Prüfung verlangt, gelten die folgenden Prüfungen. Bei einem Auftrag zur Commit- oder Release-Vorbereitung sind sie immer auszuführen.

1. Nach abgeschlossenen TypeScript-/TSX-Änderungen zuerst die schnelle Typprüfung ausführen:

   `node .\node_modules\typescript\bin\tsc --noEmit --pretty false`

   Alternativ funktioniert `corepack pnpm exec tsc --noEmit --pretty false`.

2. Nach kleinen CSS- oder Textänderungen zusätzlich ausführen:

   `git diff --check`

3. Nach größeren, routing-, build- oder release-relevanten Änderungen den vollständigen Build ausführen:

   `corepack pnpm build`

4. Richtwerte für Timeouts:
   - Typprüfung: 30 Sekunden
   - Produktions-Build: 120 Sekunden

5. Wenn ein Befehl das Zeitlimit überschreitet:
   - Nicht denselben Befehl mehrfach parallel starten.
   - Prüfen, ob versehentlich der globale `pnpm`-Wrapper verwendet wurde.
   - Den direkten TypeScript-Befehl oben als Gegenprobe verwenden.
   - Nur Prozesse beenden, die nachweislich durch die aktuelle Codex-Aufgabe gestartet wurden.

## Browser und visuelle Prüfung

- Der integrierte Browser ist in der Codex-Erweiterung für VS Code und in der Codex CLI nicht verfügbar. Das ist eine Einschränkung der Oberfläche und kein lokaler Konfigurationsfehler.
- In einer VS-Code-Sitzung nicht wiederholt versuchen, den integrierten Browser zu initialisieren.
- Für einen einfachen lokalen Erreichbarkeitstest darf bei laufendem Dev-Server beispielsweise `Invoke-WebRequest http://localhost:3000/<route>` verwendet werden. Das ersetzt keine visuelle Prüfung.
- In der Regel hat der Nutzer einen dev-server auf Port 3000 laufen, davon ist erst einmal auszugehen.
- Für echte Viewport-, Interaktions- oder Screenshot-Prüfungen das Projekt bzw. den Chat in der ChatGPT-Desktop-App öffnen und dort den integrierten Browser mit der lokalen URL verwenden.
- Browser- und Sichtprüfungen nur ausführen, wenn der Nutzer sie ausdrücklich anfordert; nicht jede Layoutkorrektur automatisch im Browser öffnen.
- Die fehlende Browser-Verfügbarkeit nur erwähnen, wenn visuelle Prüfung für das Ergebnis relevant ist. Keine wiederholten Standardhinweise bei reinem Code- oder Layout-Editing.
- Keine Browser-Test-Abhängigkeit wie Playwright hinzufügen, sofern der Benutzer nicht ausdrücklich eine projektlokale Browser-Testumgebung wünscht.

Offizielle Hinweise:

- Browser: https://learn.chatgpt.com/docs/browser

## Arbeitsverzeichnis

- Bereits vorhandene oder parallel entstandene Änderungen gehören dem Benutzer und bleiben unangetastet.
- Für Git-Befehle in dieser Windows-Umgebung bei Bedarf verwenden:

  `git -c safe.directory='C:/Users/dudoo/Downloads/bahn vibe/bahn.vibe' <befehl>`

## Zusammenarbeit und Commits

- Visuelle Änderungen zunächst uncommitted zur Prüfung durch den Nutzer lassen. Keine Commits anlegen, solange der Nutzer sie nicht ausdrücklich freigibt.
- Wenn Commits gewünscht sind, thematisch trennen und Conventional Commits verwenden, da Release Please daraus Changelog und SemVer ableitet:
  - `feat:` für neue sichtbare Funktionalität,
  - `fix:` für Korrekturen bestehenden Verhaltens oder Layouts,
  - `docs:` und `chore:` für reine Dokumentation beziehungsweise Wartung,
  - `feat!:` oder ein `BREAKING CHANGE:`-Footer nur für echte inkompatible Änderungen.
- Vor dem Commit immer `git status`, den vollständigen Diff und `git diff --check` prüfen. Keine fremden Dateien versehentlich stagen.
- Nicht nach jeder kleinen Layoutiteration bauen oder committen. Erst die Nutzerprüfung abwarten und zusammengehörige Korrekturen anschließend sinnvoll bündeln.

## Cache-Datenbank und Dokumentation

- Die Regeln dieses Abschnitts gelten für die Cache-Datenbank `connection-cache.db`, nicht für die separat bereitgestellte Direktverbindungs-Datenbank.
- `docs/database-maintenance.md` ist die verbindliche Dokumentation für Aufbau, Datenbereiche, Aufbewahrungsregeln, Migrationen und Wartungsbefehle der Cache-Datenbank.
- Bei Änderungen am Schema, an Tabellen oder Indizes, an Aufbewahrungs- und Bereinigungsregeln, am dokumentierten Runtime-Verhalten oder an Wartungsbefehlen immer prüfen, ob `docs/database-maintenance.md` angepasst werden muss, und die notwendige Dokumentationsänderung in derselben Aufgabe umsetzen. Reine interne Refactorings ohne Änderung dieses Vertrags benötigen keine Dokumentationsänderung.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sparpreis-guru/sparpreis.guru](https://github.com/sparpreis-guru/sparpreis.guru) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
