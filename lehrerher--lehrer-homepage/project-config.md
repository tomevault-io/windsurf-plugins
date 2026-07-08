---
trigger: always_on
description: This file provides context for AI assistants (e.g., Claude Code) working on this repository.
---

# CLAUDE.md — AI Assistant Guide for lehrer-homepage

This file provides context for AI assistants (e.g., Claude Code) working on this repository.

---

## Project Overview

**lehrer-homepage** is a static teacher homepage for Jan Herrmann at Oberschule Spelle (Lower Saxony, Germany). It consists of HTML pages, one CSS file, and JavaScript files — no build tools, frameworks, or package managers are used.

- **Language**: German (UI content and code comments are in German)
- **Tech stack**: HTML5, CSS3, Vanilla JavaScript (ES6+)
- **External services**: Google Fonts (CDN), Formspree (form submissions)

---

## Repository Structure

```
lehrer-homepage/
├── index.html           # Startseite: Bereiche-Übersicht, Was-ist-neu, Teaser, Kontakt
├── abgabe.html          # Student assignment submission form (Formspree-powered)
├── faecher.html         # Fächer-Übersicht (verlinkt jede fach-*.html)
├── fach-<fach>.html     # EINE öffentliche Seite pro Fach: Vorstellung + Material
│                        #   (deutsch, geschichte, wipo, informatik, werte-normen,
│                        #    mathe, englisch, sport, biologie, chemie, physik, musik,
│                        #    erdkunde, gestaltendes-werken, ag-projekte, faecheruebergreifend)
├── portal.html          # Login-geschützte Material-Schnellübersicht (Kolosseum-Redirect-Ziel)
├── kontakt.html · impressum.html · datenschutz.html
├── css/
│   └── style.css        # All site styles (inkl. globaler .mat-* / .material-karte / .fach-* Klassen)
├── js/
│   ├── main.js          # Navbar/Footer-Enhancer, Hamburger, Scroll-Spy, Footer-Jahr, Abgabe-Form
│   ├── homepage-gate.js # Auth-Gate: blendet geschützte Sektionen ein/aus
│   ├── kolosseum-login-widget.js # Login-Status-Chip in der Navbar
│   ├── kolosseum-prompt.js       # XP-Modal nach Quiz (Gast-Vorschau computeNotenpunkte)
│   ├── worksheet-xp.js           # XP-Modal/-Einlösung für Arbeitsblätter
│   ├── arena-bar.js              # Persistente Kolosseum-Statusleiste unten auf jeder Seite
│   ├── auth-guard.js             # Body verstecken bis Auth-Check (geschützte Einzelseiten)
│   ├── dynamic-content.js        # Lädt dynamische Inhalte (inhalte.json)
│   ├── materialien-dynamisch.js  # Lädt je Fach hochgeladene Materialien (Kolosseum-API, data-fach)
│   ├── was-ist-neu.js            # „Was ist neu?"-Spalten (neuigkeiten.json + inhalte.json + API)
│   ├── blog.js · blog-daten.js · blog-einreichen.js  # Blog-Rendering/-Fallback/-Upload
│   ├── supabase-config.js · supabase-leaderboard.js  # Globale Quiz-Bestenlisten
├── neuigkeiten.json     # Pflegbare Einträge für „Was ist neu?" (Funktionen)
├── inhalte.json         # Pflegbare Einträge für neue Materialien (dynamic-content.js)
├── materialien/         # Generierte interaktive Arbeitsblätter/Quizze (HTML)
│   └── _template-karten-quiz.html  # Vorlage für Bilderraten-/Karten-Quizze
├── ANLEITUNG-Bilderraten-Quiz.md   # Schritt-für-Schritt-Anleitung Bilderraten-Quiz
├── Material manuell von mir/  # Quelldateien (PPTX, Bilder, HTML-Entwürfe) von Jan Herrmann
├── upload/              # Drop folder for raw worksheet files
│   └── _erledigt/       # Processed originals (moved here after conversion)
├── regelradar.html      # RegelRadar (Rechtschreibung/Zeichensetzung): zentraler Hub
├── regelradar-selbsttest.html   # Schüler-Selbsttest (5 Varianten × 35 Items)
├── regelradar-lehrkraft.html    # Lehrkraft-Dashboard
├── regelradar-{diktat,strategien,strategieabfrage,uebungen,selbsttest2,quatsch}.html
├── reso*.html           # Reine Redirect-Stubs (alte Pfade → regelradar*.html)
├── reso-backend/        # RegelRadar-API-Backend (FastAPI + SQLite, systemd-Service)
│   ├── api.py           # Endpunkte /klasse, /klassen, /ergebnis, /ergebnisse, …
│   ├── reso-api.service # systemd-Servicedatei (Pfad bleibt /reso-api – Server-Vertrag!)
│   └── setup.sh         # Einmaliges Server-Setup-Skript
├── reso-deploy/         # Separater Deploy-Stand des Backends (eigene CLAUDE.md) – nicht das Live-Frontend
├── kolosseum/           # Lernkolosseum (geschützter Bereich, Node.js-Backend)
│   ├── public/
│   │   ├── profil.html · rangliste.html · charakter.html
│   │   ├── quiz.html · quiz-spiel.html · kampf.html · shop.html
│   │   ├── login.html · register.html
│   │   ├── js/ (main.js, arena-bar.js, avatar.js, charakter-avatar.js, site-nav.js)
│   │   └── admin/
│   │       ├── index.html · dashboard.html · quiz.html · schueler.html
│   │       ├── link-erstellen.html · material-hochladen.html · xp-vergabe.html
│   │       └── leseabenteuer.html
│   └── routes/
│       └── external.js  # XP-Berechnung (computeNotenpunkte)
└── tools/
    ├── ab_generator.py  # Watch-loop: upload/ → Claude API → materialien/ → git push
    └── requirements.txt # Python dependencies (anthropic)
```

> **Weitere eigenständige Lernseiten im Root** (öffentlich bzw. teils per `auth-guard.js`/Geheimlink
> geschützt), von den Fachseiten verlinkt: `narratologische-analyse.html`,
> `deutsch-dialektische-eroerterung.html`, `deutsch-das-parfum.html`, `deutsch-theaterprojekt.html`,
> `fuenfschrittlesemethode.html`, `stilmittel-quiz.html`, `literaturwissenschaft_quiz_v2.html`,
> `rechtschreibquiz.html`, `lernquiz_jahrgang5.html`, `ab-herrschaft-mittelalter.html`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LehrerHer/lehrer-homepage](https://github.com/LehrerHer/lehrer-homepage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
