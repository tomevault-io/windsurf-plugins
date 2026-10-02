---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

RadioSabbelNich hört mehrere Internetradio-Sender mit, schaltet bei Sprache
(Moderation/Werbung/Jingles) automatisch weiter und strahlt das Ergebnis per
Icecast neu aus. Überblick und Feature-Beschreibung: `README.md`. Wie und
warum das System so gebaut ist (Diagramme + Begründungen pro Subsystem):
`ARCHITECTURE.md` — das ist die alleinige Quelle der Wahrheit dafür, hier
in `CLAUDE.md` steht das architekturbeschreibende Wissen nicht mehr
zusätzlich.

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
  `blarks.de/radio/update/` hochladen (siehe
  `android-app/README.md`, Abschnitt "Update-Mechanismus" — sonst hält
  die App den alten Stand weiterhin für aktuell). Seit 2026-08-12
  zusätzlich unter dem festen Namen `radiosabbelnich-latest.apk`
  hochladen (dritter `scp`, siehe `android-app/README.md`/`CLAUDE.md`) —
  rein für den QR-Code im Haupt-README unten, `UpdateManager` selbst
  kennt/braucht diese Datei nicht.
- **`android-app/README.md` bei jeder inhaltlichen Änderung an der App
  nachziehen** (analog zur README-Pflicht des Docker-Projekts oben, nur
  eben für die App statt den Dienst).

Der Update-Mechanismus lief bis 2026-08-08 über einen eigenständigen
lokalen Server (`update_server.py` + systemd-Service, nur übers
Tailscale-Netz erreichbar) — seitdem stattdessen über den ganz normalen,
öffentlich erreichbaren Webserver von `blarks.de`
(`/srv/www/blarks.de/radio/update/`, statisches Verzeichnis — seit
2026-08-12 dort, davor `/srv/www/blarks.de/update_radiosabbelnich/`,
identischer Inhalt umgezogen, siehe `android-app/SESSION.md`, kein
eigener Server-Prozess mehr nötig). Bewusste Ausnahme von "Kein Auth,
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
- **Doku gehört zur Änderung, nicht danach.** Vor jedem Commit alle
  fünf Dateien nachziehen — jede hat genau eine Zuständigkeit, keine
  Doppelung zwischen ihnen:
  - **`SESSION.md`** ist append-only: pro Arbeitseinheit ein neuer Eintrag am
    Ende (Datum, Auslöser, Umsetzung, "Verifiziert" mit echten Messwerten,
    ggf. "bewusst NICHT gemacht"). Ältere Einträge werden **nicht**
    rückwirkend korrigiert — was sich später als überholt herausstellt, wird
    im neuen Eintrag richtiggestellt. Hier steht das *Wie und Warum*
    chronologisch — `ARCHITECTURE.md` unten hält nur den *aktuellen* Stand
    dieser Begründungen fest, nicht die Historie.
  - **`README.md`** beschreibt den *aktuellen Stand* für Nutzer: alles, was
    Verhalten, Setup, Bedienung, Konfigurationswerte oder die Datei-Tabelle
    verändert, muss dort mitgezogen werden (keine Historie, keine
    Doppelung von SESSION.md).
  - **`ARCHITECTURE.md`** nachziehen, wenn sich Architektur oder Invarianten
    ändern — Diagramme + die harten "warum genau so"-Begründungen pro
    Subsystem, inklusive der Liste offener Punkte am Ende. Das ist seit
    2026-08-16 die alleinige Zuständigkeit dafür (vorher stand das in
    `CLAUDE.md`).
  - **`CLAUDE.md`** (diese Datei) nachziehen, wenn sich Arbeitsabläufe,
    Konventionen oder Testmuster ändern — nicht mehr für Architektur
    (siehe `ARCHITECTURE.md` oben).
  - **`CHANGELOG.md`** (seit 2026-08-12 bei jedem Commit): verdichtete
    Ein-/Zwei-Zeiler pro nennenswerter Änderung, neueste zuerst, am
    Kopf des passenden Datumsabschnitts eingefügt (neuer Abschnitt bei
    neuem Kalendertag) — keine Begründungen/Messwerte wie in
    `SESSION.md`, nur die verdichtete Übersicht. Android-Einträge mit
    `**Android:**`-Präfix (siehe Datei-Kopf dort).
- Commit-Messages: die neueren sind Englisch, ältere Deutsch — am jeweils
  letzten Commit orientieren.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Blarks-de/radiosabbelnich](https://github.com/Blarks-de/radiosabbelnich) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
