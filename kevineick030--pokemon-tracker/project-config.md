---
trigger: always_on
description: Kevin ist Endnutzer mit ADHS und will keine Deployment-Überraschungen. Für JEDE Session (lokal wie Cloud):
---

# CLAUDE.md — Projekt-Statusdokument (Pokémon Karten Tracker)

## 🚦 Regeln für KI-Sessions (immer befolgen)
Kevin ist Endnutzer mit ADHS und will keine Deployment-Überraschungen. Für JEDE Session (lokal wie Cloud):

1. **Zuerst diese CLAUDE.md + START-HIER.md lesen.**
2. **Direkt auf `main` arbeiten — und die KI veröffentlicht selbst.** Wenn eine Änderung fertig & getestet ist, **committet und pusht die KI direkt nach `main`** (`git add -A && git commit -m "…" && git push`). Kevin muss dafür **keinen Ordner öffnen und keine `.bat` starten**. Push nach `main` = **live/produktiv** → danach in einem Satz sagen, was veröffentlicht wurde. **KEINE Preview-Branches oder Pull Requests**, außer Kevin fragt ausdrücklich danach. ⚠️ Die interaktive `deploy.bat` **nicht** selbst im Terminal ausführen (sie fragt nach Eingaben und bleibt hängen) — die ist nur Kevins Ein-Klick-Variante.
3. **Deployment:** Push nach `main` → die **GitHub Action** (`.github/workflows/deploy.yml`) macht `git pull` **und startet automatisch nur die Dienste neu** (`systemctl restart pokemon-tracker` + `pokemon-dashboard`). **Du musst nichts von Hand neu starten.** 🛑 **NIEMALS den ganzen Server rebooten** — geteilte VPS (mehrere Bots), das würde alle anderen killen; immer nur den einzelnen Dienst per `systemctl restart <name>`. Daten liegen in SQLite (`pokemon_tracker.db`).
4. **Doku aktuell halten (PFLICHT):** Nach JEDER nennenswerten Code-Änderung diese CLAUDE.md **im selben Commit** mit aktualisieren (neue Features, geänderte Abläufe, Deploy, Config). Veraltete Doku ist schlimmer als keine. Auch was Kevin als dauerhafte Regel sagt, kommt hierher – nicht nur in den Chat.
5. **Einfache Sprache, keine unnötigen Fachbegriffe.**

---

> Zweck: Damit eine **neue Session** sofort weiß, was das Projekt ist, was läuft,
> wie es betrieben/bedient wird und was noch offen ist. Bitte bei größeren
> Änderungen aktuell halten.

**Letzter Stand:** 2026-07-05 · Sprache mit dem User: **Deutsch** (Einsteiger,
nicht-technisch → einfache Schritt-für-Schritt-Erklärungen).

---

## 1. Was ist das?
Telegram-Bot + Web-Dashboard zum Tracken/Sammeln/Bewerten von Pokémon-Karten
**und versiegelten Produkten** (Tins, ETBs, Displays etc.).
Kernnutzung: **Karte/Produkt fotografieren → erkennen → Preis sehen → in
Sammlung/Watchlist legen.** Fokus: **deutsche Karten + Produkte, Cardmarket-Preise.**

---

## 2. Aktueller Stand (Ampel)

### 🟢 Funktioniert (live auf dem Server)

#### Foto-Workflow (Einzelkarten)
- Foto → Gemini-Erkennung (Name DE+EN+JP, Set, Nummer, Seltenheit, Produkttyp)
- Preis-Lookup-Kette:
  1. TCGdex → `idProduct` (Cardmarket-Produkt-ID) ermitteln
  2. **Lokaler Cardmarket Price Guide** (`cm_price_guide` SQLite-Tabelle) → `low`+`trend`+`avg7` (EUR)
  3. Fallback: TCGdex EU-Aggregate (wenn kein CM-Eintrag)
- Anzeige: `Ab: X €` (günstigstes Angebot EU-weit) + Trend + Ø7T + direkter CM-Link
- 4 Buttons: `[✅ Sammlung] [💰 Preis-Check] [🔔 Watchlist] [💼 Scalp-Track]`
  (Scalp-Track nur bei versiegelten Produkten sichtbar)
- Erkennungs-Info bleibt nach Button-Klick stehen (Buttons werden nur entfernt)

#### Cardmarket Price Guide (NEU, 2026-06-04)
- **Datei:** `https://downloads.s3.cardmarket.com/productCatalog/priceGuide/price_guide_6.json`
- **75.099 Pokémon-Produkte** mit low/trend/avg/avg7/avg30 (EUR)
- **Täglich 06:00** automatisch heruntergeladen + in SQLite (`cm_price_guide`) importiert
- Kein API-Key nötig. Lookup: `idProduct` → sofortige lokale DB-Abfrage (kein Rate-Limit)
- Modul: `cm_priceguide.py` (download_and_import, get_price, is_ready)

#### TCGdex-Lookup (zwei-Pfad-Strategie, NEU 2026-06-04)
- **Pfad 1 (bevorzugt):** Set + Kartennummer → direkter TCGdex-Endpunkt `/{lang}/cards/{set_id}-{num}`
  → immer exakt, kein Scoring-Fehler möglich. Plausibilitätscheck: Basis-Pokémon-Name.
- **Pfad 2 (Fallback):** Namenssuche → Nummer dominiert Scoring (+10 Treffer / -20 Mismatch).
  Wenn kein Kandidat die Nummer trifft → `None` statt falscher Karte.
- **Prinzip: lieber kein Preis als ein Preis von der falschen Karte.**

#### Sammlung (Einzelkarten + versiegelte Produkte)
- Foto → ✅ Sammlung → Kaufpreis eingeben → Eintrag in `portfolio`-Tabelle
- Funktioniert für Einzelkarten UND Tins/ETBs/Displays
- **Tägliche Neubewertung (06:10, direkt nach Price-Guide-Download):**
  `portfolio.update_all_values()` → TCGdex→CM-Price-Guide
- `/wert`, `/sammlung`, `/gekauft`

#### Japanische Karten
- Gemini-Prompt verlangt `card_name_en` als **PFLICHT** (auch bei JP-Karten, mit Beispielen)
- TCGdex EN-Lookup liefert `idProduct` → CM Price Guide-Lookup funktioniert
- Fallback: Namens-Suchlink wenn idProduct nicht gefunden

#### Weitere funktionsfähige Features
- **`/preis <name>`** (TCGdex-Preise), **KI-Chat** (Claude Haiku, Freitext)
- **Budget** (`/budget`, `/ausgabe`), **Watchlist** (`/add`, `/watchlist`)
- **Releases** (`/releases`, `/release_add`)
- **Web-Dashboard** (Sammlung-Galerie, Wert, Watchlist, Budget, Scalp, Releases)
  URL: `http://87.106.255.195:8090`

### 🟡 Eingeschränkt
- **Preise sind EU-weite Cardmarket-Durchschnitte** (kein DE-Filter): `low` = günstigstes
  EU-Angebot, nicht nur Deutschland. Der Link führt zu allen Cardmarket-Angeboten.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kevineick030/pokemon-tracker](https://github.com/kevineick030/pokemon-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
