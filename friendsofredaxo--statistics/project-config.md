---
trigger: always_on
description: Ein REDAXO-Addon zur serverseitigen Erfassung und Auswertung von Seitenaufrufen, Besuchern, Medien-Downloads, Referrern, Events, Bots und Session-Metriken. Die Backend-Auswertung basiert auf ECharts und DataTables.
---

# Statistics Addon - REDAXO

Ein REDAXO-Addon zur serverseitigen Erfassung und Auswertung von Seitenaufrufen, Besuchern, Medien-Downloads, Referrern, Events, Bots und Session-Metriken. Die Backend-Auswertung basiert auf ECharts und DataTables.

## Tech Stack

- Sprache: PHP
- CMS: REDAXO 5
- Frontend/Backend-Visualisierung: ECharts, DataTables, jQuery, Bootstrap-Collapse im REDAXO-Backend
- Datenbank: MySQL/MariaDB
- Namespace: `AndiLeni\Statistics`

## Projektstruktur

```text
statistics/
├── boot.php                          # Asset-Registrierung, Hooks, Tracking-Aktivierung
├── install.php                       # Datenbankschema und manuelle Zusatzindizes
├── update.php                        # Update-Handler
├── uninstall.php                     # Cleanup
├── README.md                         # Nutzer- und Architektur-Dokumentation
├── AGENTS.md                         # Projektwissen für Agenten
├── assets/
│   ├── statistics.js                 # DateFilter, DataTables, Lazy Loading
│   ├── statistics.css                # Backend-Styles
│   └── vendor/                       # ECharts, DataTables, Themes
├── fragments/
│   ├── filter.php                    # Datumsfilter
│   ├── overview.php                  # Kennzahlen oben auf stats
│   ├── main_chart.php                # Tages-/Monats-/Jahrescharts
│   ├── data_vertical.php             # Standardlayout für Chart + Tabelle
│   └── collapse.php                  # Einklappbare Tabellen, optional lazy
├── lib/
│   ├── ChartData.php                 # Hauptchart-Aggregationen
│   ├── StatsChartConfig.php          # Renderer/Builder für zentrale Chart-Optionen per JSON-Script
│   ├── StatsDashboard.php            # Komposition der stats-Startseite (Fragmente + Page-Config)
│   ├── StatsMainChartSection.php     # Renderer für den Hauptchart-Bereich der stats-Seite
│   ├── StatsLazySection.php          # Renderer für die unteren Lazy-Placeholder der stats-Seite
│   ├── StatsSubpageRenderer.php      # Gemeinsame Renderer für Datumsfilter und Sections auf Unterseiten
│   ├── ListData.php                  # Tabellen unter den Hauptcharts
│   ├── Summary.php                   # Overview-Kennzahlen
│   ├── Pages.php                     # Seiten-Auswertungen
│   ├── PageDetails.php               # URL-Details
│   ├── RefererDetails.php            # Referrer-Details
│   ├── StatsLazyBlockRenderer.php    # Renderer für lazy geladene Blöcke
│   ├── rex_api_statistics_lazy_block.php # REDAXO-API für Lazy Loading
│   ├── api/                          # Event-API und Detailklassen
│   ├── media/                        # Medien-Detailklassen
│   └── data/                         # Browser, OS, Country, Sessionstats etc.
└── pages/
    ├── stats.php                     # Hauptdashboard
    ├── pages.php                     # URL-Übersicht
    ├── referer.php                   # Referrer-Übersicht
    ├── media.php                     # Medien-Übersicht
    └── events.php                    # Event-Übersicht
```

## Coding Conventions

- Bestehenden Stil im Addon beibehalten, auch wenn er historisch nicht überall vollständig vereinheitlicht ist.
- Kleine, gezielte Änderungen bevorzugen. Keine großflächigen Umbauten ohne klaren Performance- oder Wartungsvorteil.
- Bei Backend-Ausgabe HTML möglichst direkt und einfach halten. Viele Bereiche rendern Tabellen bewusst manuell statt über `rex_list`.
- Änderungen an großen Statistikseiten zuerst auf gemeinsame Datenquellen und doppelte SQL-Abfragen prüfen.

## Wichtige Architekturregeln

### Performance zuerst an der Datenquelle lösen

Wenn eine Seite langsam ist, zuerst diese Punkte prüfen:

1. Fehlen passende Indizes?
2. Wird dieselbe Aggregation mehrfach abgefragt?
3. Wird ein Block bereits gerendert, obwohl er initial nicht sichtbar ist?
4. Wird `rex_list` für große Aggregationen verwendet, obwohl eine direkte Tabelle günstiger wäre?

### Prefix-Indizes für lange Spalten

- URL- und Referrer-Spalten sind lang genug, dass normale Vollindizes unter `utf8mb4` scheitern können.
- Diese Indizes werden in `install.php` per SQL und mit Prefix-Länge angelegt.
- Vor Änderungen an diesen Indizes immer die MySQL-/MariaDB-Key-Length-Grenzen mitdenken.

### Startseite `pages/stats.php`

Die Statistik-Startseite ist absichtlich in synchron und lazy getrennt:

- synchron: Filter, Overview, Hauptcharts, Heatmap
- lazy beim Tab-Wechsel: Monats- und Jahreschart
- lazy bei Sichtbarkeit: Geräteblock, erweiterte Session-/Länder-Statistik, Bot-Tabelle
- lazy bei Benutzeraktion: Tabellen unter den Hauptcharts erst beim Aufklappen

Neue große Sektionen auf dieser Seite sollten standardmäßig nicht direkt synchron gerendert werden.

Die Seitenkomposition selbst läuft über `StatsDashboard.php`. `pages/stats.php` sollte möglichst nur noch Daten beschaffen und die Helfermethoden für Filter, Overview, Hauptchart-Bereich, Lazy-Placeholder und JSON-Konfiguration aufrufen.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FriendsOfREDAXO/statistics](https://github.com/FriendsOfREDAXO/statistics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
