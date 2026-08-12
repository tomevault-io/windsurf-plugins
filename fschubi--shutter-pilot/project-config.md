---
trigger: always_on
description: Diese Datei führt Claude (Entwickler-KI) als eigene Projektdokumentation:
---

# CLAUDE.md – Arbeitsgrundlage & Fortschritts-Doku

Diese Datei führt Claude (Entwickler-KI) als eigene Projektdokumentation:
Funktionsumfang, Konventionen und ein fortlaufendes Fortschritts-Log. Sie wird
bei jedem größeren Arbeitsschritt aktualisiert und mit gepusht.

## Projekt

**Shutter Pilot** – Custom Integration für Home Assistant, die Rollläden nach
Zeit, Helligkeit oder Sonnenstand fährt. Konfiguriert wird nicht über
YAML oder Options-Flow, sondern über ein **eigenes Sidebar-Panel**.

- Repo: https://github.com/fschubi/shutter_pilot (Branch `master`)
- Verteilung über HACS · Mindestversion Home Assistant 2024.6.0
- `single_config_entry: true` – es gibt genau einen Config-Entry
- Sprache im Projekt: **Deutsch** (Commits, Changelog, Kommentare, Forumstexte).
  Code-Bezeichner und Docstrings im Python-Teil sind englisch.

## Aufbau

```
custom_components/shutter_pilot/
  __init__.py        Setup, Panel-Registrierung, WebSocket-API, Minutentakt
  const.py           Alle Config-Keys, Defaults, Events – die Referenz
  helpers.py         Herzstück: Beschattungslogik, Positionen, Sperren (~980 Z.)
  scheduler.py       Zeit- und Sonnenmodus: Fahrten planen und auslösen
  brightness.py      Helligkeitsmodus mit erlaubten Zeitfenstern
  elevation.py       Beschattung: Elevation, Azimut, Bedingungen, pro Rollladen
  ventilation.py     Automatisches Lüften nach Bedingungen
  awning_guard.py    Markisen: Wind-, Regen- und Frostschutz (Sperre + Zwangsfahrt)
  schedule_times.py  Zeitmathematik: Wochenende, Jitter, Zeitklammern
  window_trigger.py  Reaktion auf Fensterkontakte
  window_helper.py   Fensterzustand und Aussperrschutz
  cover_tracker.py   Positionen mitschreiben, nach Neustart wiederherstellen
  cover_verify.py    Fahrtkontrolle: erreicht der Rollladen die Position?
  position_store.py  JSON-Speicher der letzten Positionen
  weather_data.py    Tagesvorhersage über weather.get_forecasts
  group_actions.py   Folgeaktion Licht je Bereich
  switch/sensor/binary_sensor.py   Entitäten
  services.py        Dienste (Gruppenaktionen)
  frontend/shutter-pilot-panel.js  Das komplette Panel (~2560 Z., ein File)
tests/               pytest-Suite (551 Tests)
```

## Funktionsumfang

### Bereiche und Modi

Ein **Bereich** bündelt Rollläden und legt fest, *wann* gefahren wird. Jeder
Rollladen hat einen Bereich fürs Hochfahren und einen fürs Runterfahren – die
dürfen verschieden sein (morgens raumweise, abends alle zusammen).

| Modus | Auslöser |
| --- | --- |
| `time` | feste Uhrzeiten, getrennt für Woche und Wochenende |
| `brightness` | Helligkeitssensor mit Schwellen, nur in erlaubten Zeitfenstern |
| `sun` | Sonnenauf-/-untergang plus Offset, optional in Zeitklammern |

Wochenendwerte fallen immer auf die Wochentagswerte zurück, wenn sie leer
bleiben. Statt Samstag/Sonntag kann ein **Workday-Sensor** entscheiden
(Feiertage, Schichtdienst). Eine **Präsenzsimulation** streut die Zeiten um bis
zu N Minuten; der Wert ist pro Tag stabil, nicht pro Fahrt.

### Rollläden

Je Rollladen: Positionen für offen, geschlossen und Sonnenschutz, optional eine
**abweichende Schließposition** für laue Abende und eine **Frostposition**,
optional **Lamellenwinkel** (Raffstore). Fenstersensor mit Aussperrschutz, dazu
optional ein zweiter Kontakt, wenn „gekippt" als eigene Entität gemeldet wird,
und eine **Entprellung** (0–30 s), bevor auf „geschlossen" reagiert wird.

### Beschattung

Aktiv, wenn **alle** Bedingungen zugleich gelten:

1. Sonnenhöhe im konfigurierten Bereich (min/max)
2. Sonne steht vor den Fenstern (Azimut, Kompass-Schnellwahl)
3. bis zu **vier Zusatzbedingungen** – Binärsensor, Zahlenwert mit Ein-/
   Ausschaltschwelle oder Textzustand (Wetterlage). Dieselbe Mechanik trägt die
   eigenen Slots `close`, `frost` und `vent_a`/`vent_b` – die sind aber **fail
   closed**, ein toter Sensor löst dort nichts aus.
4. Datum liegt im konfigurierten **Beschattungszeitraum** (Jahreswechsel möglich)
5. Uhrzeit liegt im **Beschattungs-Zeitfenster** (`shade_from`/`shade_to`, beide
   einzeln optional, je Bereich und je Rollladen). Die einzige der fünf
   Prüfungen, die nicht die Sonne beschreibt, sondern den Haushalt – und
   bewusst **ohne** Wrap über Mitternacht

Geometrie und Bedingungen lassen sich **pro Rollladen** überschreiben. Der
Rückfall wirkt je Bedingungs-Slot: gesetzt am Rollladen ersetzt den Slot,
leer erbt den Bereichswert. Die Hysterese liegt deshalb pro Cover, nicht pro
Bereich – sonst hebt eine Wolke vor einem Fenster die Beschattung des anderen
auf. Ein fehlender oder toter Sensor blockiert nie (fail open).

### Markisen

Ein Eintrag in derselben `shutters`-Liste, unterschieden durch `device_kind`
(fehlt = Rollladen, keine Migration). Der Fahrweg ist **derselbe**: die
Beschattung fährt auf `position_sun_protect` und gibt auf `position_open` frei,
und welche Zahl das ist, entscheidet die Konfiguration – bei der Markise 100
bzw. 0. Nur die *Rückfallwerte* in `get_position_for_role()` drehen sich.

Anders sind zwei Dinge:

1. **Kein Zeitplan.** Scheduler, Helligkeit, Lüften und Fenstertrigger filtern
   Markisen aus (`only_shutters()`), der Aussperrschutz steigt früh aus – er
   klemmt nach unten, und unten ist bei einer Markise die *sichere* Seite.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fschubi/shutter_pilot](https://github.com/fschubi/shutter_pilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
