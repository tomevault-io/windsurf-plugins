---
trigger: always_on
description: **Fennek** — Multi-App-Handheld-Firmware für das **LilyGO T-Deck Pro V1.1** (ESP32-S3, 8 MB PSRAM), PlatformIO/Arduino. Remote `danst0/fennek`. Version: `FENNEK_VERSION` in `src/config.h`, Log-Präfix `[FENNEK]`, NVS-Namespace `fennek`, SD-Cache `/.fennek`. Apps: Musik, Hörbuch, Lesen, Mesh, Spiele, Dateien, Notizen, Wecker, Karten, Kopfrechnen, Karteikarten, Kalender, Todo. Code/Doku/Serial-Ausgaben **deutsch**; Textausgaben über `gui::print` (UTF-8 → CP437).
---

# CLAUDE.md

## Projekt

**Fennek** — Multi-App-Handheld-Firmware für das **LilyGO T-Deck Pro V1.1** (ESP32-S3, 8 MB PSRAM), PlatformIO/Arduino. Remote `danst0/fennek`. Version: `FENNEK_VERSION` in `src/config.h`, Log-Präfix `[FENNEK]`, NVS-Namespace `fennek`, SD-Cache `/.fennek`. Apps: Musik, Hörbuch, Lesen, Mesh, Spiele, Dateien, Notizen, Wecker, Karten, Kopfrechnen, Karteikarten, Kalender, Todo. Code/Doku/Serial-Ausgaben **deutsch**; Textausgaben über `gui::print` (UTF-8 → CP437).

- Aktiver Code `src/`, einzige Build-Env `fennek`. `lib/meshcore/` + `lib/ed25519/` = vendored MeshCore. Alte Meck-Firmware in der Git-Historie.
- `boards/t-deck_pro.json`, Partition `default_16MB.csv` (6,5 MB App-Slots, 3,4 MB SPIFFS, 20 KB NVS).

## Build & Flash

```bash
pio run -e fennek              # bauen
pio run -e fennek -t upload    # flashen (/dev/ttyACM0, 921600 Baud)
pio device monitor -b 115200  # Boot-Log + Exception-Decoder
```

Keine Unit-Tests; Verifikation am Gerät. Bootet vollständig **ohne SD-Karte**.

Debug-Flags (`-D`): `AUDIO_DEBUG_GAP`, `PLAYLIST_SELFTEST`, `MESH_SMOKE_TEST`, `GAMES_SMOKE_TEST`, `SLEEP_WAKE_TEST`, `BATTLOG`.

## Toolchain — nicht aktualisieren

`espressif32@6.11.0` (Arduino-ESP32 2.0.17 / IDF 4.4), `ESP32-audioI2S#2.0.6`, RadioLib ^7.3 (`RADIOLIB_GODMODE=1`), rweather/Crypto, densaugeo/base64. Upgrades nur auf ausdrücklichen Wunsch.

## Architektur — Anti-Stotter-Invarianten

**E-Ink, SD und LoRa-SX1262 teilen denselben SPI-Bus** (HSPI, SCLK 36/MOSI 33/MISO 47).

```
Core 0:  Audio-Decode-Task (Prio 4) ── audio.loop()
Core 1:  Arduino loop() ── appmgr ── g_spiMutex (core/board.h)
Core 1:  Mesh-Pumpe (mesh_app::background → spiLock)
```

1. **Jeder SPI-Zugriff (SD, E-Ink, LoRa) unter `spiLock()`/`spiUnlock()`.**
2. **`audio.*`-Aufrufe nur im Audio-Task** — UI sendet Kommandos via Queue (`services/audio.cpp`).
3. **SPI-Mutex-Freigabe während E-Ink-BUSY** (`setBusyCallback` in `core/display.cpp`).
4. **256 KB PSRAM-Read-Ahead** (`audio.setBufsize(-1, 256*1024)`).
5. **SPI auf 4 MHz** (`SPI_BUS_HZ`) — 8 MHz korrumpiert SD-Writes.
6. **LoRa-CS (GPIO 3) im Leerlauf HIGH**; Radio nutzt bestehende `g_spi`-Instanz (`P_LORA_SCLK` NICHT definieren).
7. **E-Ink-Refresh nur bei Änderung** (Dirty-Flag); Fortschritt/Statuszeile nur als `renderRegion`.
8. **NVS für Settings/Bookmarks** (nie SPI-Bus). SD für Bulk-Caches + Mesh-Daten. Mesh-Identity auf SPIFFS.
9. **WiFi aktiv ⇒ `audio::stop()` + `mesh_client::setSuspended(true)`** (webfm-Regel — alle WLAN-Dienste).

## Module (`src/`)

- `config.h` — Pins & Konstanten.
- `core/board.*` — SPI-Bus, SD-Mount, `g_spiMutex`, `loraPower`, `gpsPower`. GPS (GPIO39) + DRV2605 (GPIO2) beim Boot aus. Kein 4G-Modem (diese Variante = Audio, PCM5102A-DAC).
- `core/display.*` — GxEPD2-E-Ink, `render(fn,full)` / `renderRegion(fn,y,h)`, BUSY-Callback.
- `core/gui.*` — `toCp437`/`print`/`printAt`/`textBounds`, `drawButton`, `drawRowText`, `Rect`.
- `core/touch.*` + `core/hyn/` — CST328 (I2C 13/14). **hyn/ nicht refactoren.**
- `core/keyboard.*` — TCA8418 (I2C 0x34), Sticky Shift/Alt/Sym, Key-Repeat 400/150 ms.
- `core/battery.*` — BQ27220-Fuel-Gauge (nur Lesen).
- `core/settings.*` — NVS: Lautstärke, letzte App, Resume-Positionen, Bookmarks. **WLAN-Profile** (bis 20) als kompakter Blob `wifis` (Migration aus Alt-Keys `wssid`/`wpass` = Slot 0); `wifiCount/wifiSsidAt/wifiPassAt/wifiSet/wifiRemove`, Slot 0 = Legacy-Einfachzugriff (`wifiSsid`/`setWifiSsid`, ini-Export/webfm).
- `core/console.*` — USB-Debug-Konsole (`help`). Befehle: `status`, `time`/`tz`, `mesh init`, `mesh eco`, `advert [flood]`, `public`/`join`/`chan`/`dm`, `pos`, `gps [scan|off]`, `wifi`, `alarm`, `ollama`, `podcast`, `ota`, `todo`, `cal`, `calibre`, `nav`, `scrobble`, `notes`, `rm`. Konsolen-Eingabe verlängert Auto-Standby (`power::noteActivity()`).
- `core/power.*` — Knopf (kurz=Tastensperre, lang=Standby), Auto-Standby, Deep Sleep. **CPU-Takt-Governor:** Basistakt 80 MHz ab `boostBegin()` (Setup-Ende); `boostLock()/boostUnlock()` (Zähler) → 240 MHz bei Audio-Decode, WLAN (zentral via WiFi-Events — Services müssen nichts tun), Schach-KI, EPUB-Konvertierung, Konsolen-Befehlen. APB bleibt 80 MHz, SPI/UART/I2S unberührt. **Zwei Seitenknöpfe: OBEN=GPIO0 (Firmware), UNTEN=RST (Hardware-Reset — sofort, Firmware sieht nichts).** Timer-Wake: Minimal-Pfad (nur Banner), alle 12 Wakes Vollbild. Wecker-Wake: `handleTimerWake()` gibt `false` → Vollboot. Schutznetze (nach 3 stillen Tiefentladungen 06-07/26): Task-WDT um Minimal-Pfad + `enterStandby()`, Timeout der Loslass-Warteschleife, `noteBoot()`-Crash-Loop-Bremse (3 abnormale Resets → Not-Standby nur Knopf-Wake), Tiefentladeschutz (≤5 % → Standby ohne Stunden-Wake), Schlaf-Diagnose im RTC-RAM (`logSleepDebug()` kippt Phase + Wake-Historie ins battlog).
- `core/appmgr.*` — App-Lifecycle, Statuszeile, Dirty-Koaleszenz, 30-s-Persistenz. Tap Statuszeile = Home.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danst0/fennek](https://github.com/danst0/fennek) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
