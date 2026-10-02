---
trigger: always_on
description: Lies diese Datei ZUERST, bevor du irgendwas am Code änderst. Sie enthält die
---

# Eiswolfs Flightradar (CYD) — Projektkontext für Claude Code

Lies diese Datei ZUERST, bevor du irgendwas am Code änderst. Sie enthält die
Dinge, die man wissen muss, um nicht dieselben Fehler nochmal zu machen, die
in der Entwicklung schon mal aufgetreten und behoben wurden.

## Was ist das Projekt?

Ein Live-ADS-B-Flugradar auf einem ESP32 "Cheap Yellow Display" (CYD,
ESP32-2432S028), 240x320 Touch-TFT. Zeigt Flugzeuge in der Nähe auf einem
rotierenden Radarschirm, mit Detail-Panel (Modell, Höhe, Speed, Route,
Squawk), Näherungs-LED-Alarm, WLAN-Verwaltung (bis zu 3 Netzwerke),
Standort-Presets (auch für fremde Orte weltweit), Airline-Filter, Flugbuch
mit Statistik, und Menüs/Splash-Screen mit einer dezenten twinkelnden
Sterne-Animation.

Aktuelle Versionsnummer: siehe `Config::APP_VERSION` in `src/config.h`
(dort immer aktuell, hier bewusst nicht mehr hart eingetragen, damit diese
Datei nicht wieder veraltet). Öffentliches Repo:
https://github.com/Eiswolf-BG/eiswolfs-flightradar-CYD

## Wer nutzt das?

Alex — kompletter Anfänger bei Arduino/Embedded-Entwicklung, arbeitet mit
VS Code + PlatformIO auf einem Mac. Bitte auf Deutsch antworten. Erklärungen
gerne etwas ausführlicher, nicht von Fachbegriffen ausgehen, die als bekannt
vorausgesetzt werden.

## Tech-Stack

- PlatformIO, `platform = espressif32`, `board = esp32dev`, `framework = arduino`
- TFT_eSPI (Display), XPT2046_Touchscreen (Touch), ArduinoJson, TinyGPSPlus
- Dual-Core-Design: Netzwerk (WLAN, ADS-B-Polling) auf Core 0, Display/Touch
  auf Core 1 — die UI blockiert nie durch Netzwerk-Requests.
- Daten: [adsb.lol](https://adsb.lol) (Flugzeugpositionen, `api.adsb.lol`),
  [hexdb.io](https://hexdb.io) (Modell-Lookups), [ip-api.com](https://ip-api.com)
  (IP-Geolocation)
  - Bis vor Kurzem wurde adsb.fi genutzt - Wechsel zu adsb.lol, weil adsb.fi
    dem ESP32-Client trotz gueltigem HTTP 200 und validem JSON dauerhaft
    leere Flugzeuglisten lieferte (curl von demselben Netzwerk aus lieferte
    jederzeit volle Daten) - vermutlich Cloudflare-Bot-Management/TLS-
    Fingerprinting gegen den mbedTLS-Client des ESP32. adsb.lol hat
    identisches URL-/JSON-Schema (nur `/v2/...` statt `/api/v3/...`,
    Feldnamen unveraendert) und war ohne Parser-Anpassung nutzbar. Falls das
    Thema nochmal auftaucht: siehe `Config::ADSB_API_HOST`/
    `Config::ADSB_HTTP_TIMEOUT_MS` in `config.h` sowie `adsb_client.cpp`.

## ⚠️ WICHTIGSTE FALLE: Der eigene Font ist grundlinien-verankert

Die eingebauten TFT_eSPI-Fonts (GLCD, Font2 etc.) sind reines ASCII. Für
Umlaute/Akzente (Deutsch, Türkisch, Französisch, Spanisch, Italienisch,
brasilianisches Portugiesisch, Niederländisch)
gibt's einen **selbst generierten Font** (`src/ui_font.h`, via
`tft.setFreeFont(&UiFont11pt)` global in `main.cpp::setup()` aktiviert, gilt
danach für JEDEN `print()`/`drawString()`-Aufruf in der ganzen App).

**Der entscheidende Unterschied zu den eingebauten Fonts:** Bei
`setCursor(x, y); print(...)` ist `y` bei unserem Font die **Grundlinie**
(Baseline), NICHT die obere Kante wie beim alten GLCD-Font. Der Text wächst
von `y` aus nach OBEN (um den Ascent, ca. 9px bei Size 1, ca. 16-18px bei
Size 2), nicht nach unten.

**Das hat in der Vergangenheit zu folgenden Bugs geführt (alle behoben, aber
Vorsicht bei neuem Code!):**
- Zu kleine y-Werte (z.B. `setCursor(10, 2)`) → Text ragt oben aus dem
  Bildschirm/Container heraus oder wird abgeschnitten. **Faustregel:
  y sollte bei Size 1 nie kleiner als ~14 sein, bei Size 2 nie kleiner als
  ~24-26 (abhängig vom Container).**
- Eingabefelder/Boxen: Baseline muss nahe der UNTERKANTE der Box liegen,
  nicht in der Mitte wie man's vom alten Font gewohnt wäre.
- Zwei Textgrößen kurz hintereinander in einem eng bemessenen Layout
  (Label Size 1 direkt über Wert Size 2) sind fehleranfällig — im
  Statistik-Screen deshalb bewusst auf EINHEITLICHE Größe (nur Farbe
  unterscheidet Label/Wert) umgestellt, das ist robuster.
- `drawString()` mit `setTextDatum(MC_DATUM)` (zentriert) ist NICHT
  betroffen — TFT_eSPI rechnet die Zentrierung selbst korrekt aus, egal ob
  Baseline- oder Top-verankert. Nur rohes `setCursor()`+`print()` ist die
  Gefahrenzone.
- Langer, mehrzeiliger Text (z.B. Erklärtexte) NIE mit TFT_eSPI's
  eingebautem Auto-Wrap verlassen — das bricht mitten im Wort ab. Stattdessen
  den `layoutWrapped()`-Helper verwenden (in `location_presets_screen.cpp`,
  `wifi_manage_screen.cpp` und `radar_screen.cpp` je einmal implementiert,
  macht wortweisen Umbruch anhand echter Pixel-Breite, meist plus
  optionales Scrollen). **NIE** eine Variante verwenden, die Text nur bis
  zu einem festen Zeilenlimit umbricht, ohne JEDE resultierende Zeile
  erneut auf ihre tatsächliche Pixel-Breite zu prüfen (Beispiel für genau
  diesen Fehler: `drawWrappedCenteredHint()` in `radar_screen.cpp`, fest
  auf höchstens 2 Zeilen begrenzt — bei einem längeren deutschen Text im
  Höhen-Farben-Legende-Overlay lief die Zeile dadurch links UND rechts
  über den Bildschirmrand hinaus, siehe Git-Historie/Bugfix). Ein
  festes Zeilenlimit ist nur dann unkritisch, wenn zusätzlich VORHER
  geprüft wird, dass der Text in ALLEN 8 Sprachen tatsächlich in dieses
  Limit passt — im Zweifel immer das uneingeschränkte `layoutWrapped()`
  nehmen.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eiswolf-BG/eiswolfs-flightradar-CYD](https://github.com/Eiswolf-BG/eiswolfs-flightradar-CYD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
