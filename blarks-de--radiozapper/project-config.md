---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

RadioSabbelNich hört mehrere Internetradio-Sender mit, schaltet bei Sprache
(Moderation/Werbung/Jingles) automatisch weiter und strahlt das Ergebnis per
Icecast neu aus. Überblick und Feature-Beschreibung: `README.md`.

## Android-Prototyp (separates Projekt)

`android-app/` ist ein eigenständiges natives Kotlin/Gradle-Projekt (media3 +
Vosk-Android), das dasselbe Grundprinzip lokal auf dem Handy nachbildet — ein
komplett anderer Tech-Stack, eigene Toolchain, eigenes `README.md` dort. Es
läuft unabhängig von der hier beschriebenen Docker-Instanz und ändert nichts
an deren Architektur/Verhalten; die obigen Konventionen (Deutsch, SESSION.md,
VERSION-Pflege) gelten für den Docker-Dienst, nicht 1:1 für den Android-Code.
Seit 2026-08-07 wird es aber mitgepflegt, dafür zwei feste Regeln:

- **Nach jedem Android-Build** die entstandene Debug-APK lokal nach
  `android-app/radiosabbelnich.apk` kopieren (fester, einfach auffindbarer
  Pfad statt des tief verschachtelten
  `app/build/outputs/apk/debug/app-debug.apk` — letzterer ist ohnehin
  gitignored) UND zusätzlich mit Zeitstempel im Dateinamen
  (`radiosabbelnich-YYYYMMDD-HHMMSS.apk`) sowie ein passendes
  `version.json` (`{"buildTime": "...", "apkFile": "..."}`) nach
  `blarks.de/update_radiosabbelnich/` hochladen (siehe
  `android-app/README.md`, Abschnitt "Update-Mechanismus" — sonst hält
  die App den alten Stand weiterhin für aktuell).
- **`android-app/README.md` bei jeder inhaltlichen Änderung an der App
  nachziehen** (analog zur README-Pflicht des Docker-Projekts oben, nur
  eben für die App statt den Dienst).

Der Update-Mechanismus lief bis 2026-08-08 über einen eigenständigen
lokalen Server (`update_server.py` + systemd-Service, nur übers
Tailscale-Netz erreichbar) — seitdem stattdessen über den ganz normalen,
öffentlich erreichbaren Webserver von `blarks.de`
(`/srv/www/blarks.de/update_radiosabbelnich/`, statisches Verzeichnis,
kein eigener Server-Prozess mehr nötig). Bewusste Ausnahme von "Kein Auth,
nur hinter VPN" unten: verteilt nur eine App-Binary ohne Nutzerdaten,
Details und Abwägung in `android-app/SESSION.md`.

## Sprache und Konventionen

- **Alles auf Deutsch**: Kommentare, Docstrings, Log-Meldungen, UI-Texte,
  README, SESSION.md. Neue Beiträge genauso.
- Kommentare erklären **warum**, nicht was — insbesondere bei allem, wo
  ein naheliegender Ansatz nachweislich nicht funktioniert hat (z.B. warum
  `_write()` in `stations_store.py` kein write-temp-then-rename macht, warum
  der Import-Check nicht per ffprobe läuft). Diese Begründungen sind hart
  erarbeitet; nicht wegkürzen.
- **Doku gehört zur Änderung, nicht danach.** Vor jedem Commit beide Dateien
  nachziehen:
  - **`SESSION.md`** ist append-only: pro Arbeitseinheit ein neuer Eintrag am
    Ende (Datum, Auslöser, Umsetzung, "Verifiziert" mit echten Messwerten,
    ggf. "bewusst NICHT gemacht"). Ältere Einträge werden **nicht**
    rückwirkend korrigiert — was sich später als überholt herausstellt, wird
    im neuen Eintrag richtiggestellt. Hier steht das *Wie und Warum*.
  - **`README.md`** beschreibt den *aktuellen Stand* für Nutzer: alles, was
    Verhalten, Setup, Bedienung, Konfigurationswerte oder die Datei-Tabelle
    verändert, muss dort mitgezogen werden (keine Historie, keine
    Doppelung von SESSION.md).
  - **`CLAUDE.md`** (diese Datei) nachziehen, wenn sich Architektur,
    Invarianten oder Arbeitsabläufe ändern — inklusive der Liste offener
    Punkte unten.
- Commit-Messages: die neueren sind Englisch, ältere Deutsch — am jeweils
  letzten Commit orientieren.
- **Versionspflege (seit 2026-08-06)**: `VERSION` am Repo-Root, Format
  `vMAJOR.MINOR.PATCH build YYYY-MM-DD HH:MM Uhr`. Start war `v1.0.0`.
  Jede Änderung, die committet wird, erhöht PATCH um `+0.0.1` und trägt
  Datum/Uhrzeit des Commits nach, bis der Nutzer explizit etwas anderes
  vorgibt (z.B. einen MINOR/MAJOR-Sprung). Vor jedem Commit prüfen/
  nachziehen, wie bei SESSION.md/README.md oben.

## Betrieb und Deployment

Es gibt **kein Test-Framework, keine Linter-Config, keine CI**. Verifikation
läuft über die unten beschriebenen manuellen Muster und wird in SESSION.md
protokolliert.

```bash
docker compose up -d --build radiosabbelnich   # bauen + neustarten (Standard-Zyklus)
docker compose logs -f radiosabbelnich         # Konsole: nur Ereignisse (INFO)
tail -f data/logs/radiosabbelnich.log          # Volles DEBUG-Log, überlebt Neustarts
```

Ein frischer Clone braucht `cp env.example .env` **und `touch data/fingerprints.db`**:
die DB ist als einzelne Datei gebindmountet und gitignored — fehlt sie, legt
Docker ein Verzeichnis an und SQLite scheitert in einer Neustartschleife.

Das Web-Interface läuft auf Port 5000, Icecast auf 8000 (siehe `.env`).
Änderungen an `stations.json`/`settings.json` wirken **ohne Neustart** (der
Hauptloop lädt neu), Code-Änderungen brauchen einen Rebuild.

### Testen ohne das laufende Deployment anzufassen

Bewährtes Muster (in SESSION.md mehrfach dokumentiert): `*.py` in ein
Temp-Verzeichnis kopieren, dort eine eigene `stations.json`/`settings.json`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Blarks-de/radiozapper](https://github.com/Blarks-de/radiozapper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
