---
trigger: always_on
description: ESP8266/ESP32-based Message Board firmware (`rda_msg_board`). Controls MAX7219 LED matrices to display scrolling messages. Features a web interface, HTTP REST API, MQTT support, clock with multiple font faces, recurrent alarms, sleep mode, and Home Assistant MQTT Discovery integration. Originally developed for Arduino IDE, migrated to PlatformIO.
---

# GEMINI.md - Project Context & Rules

## Project Description
ESP8266/ESP32-based Message Board firmware (`rda_msg_board`). Controls MAX7219 LED matrices to display scrolling messages. Features a web interface, HTTP REST API, MQTT support, clock with multiple font faces, recurrent alarms, sleep mode, and Home Assistant MQTT Discovery integration. Originally developed for Arduino IDE, migrated to PlatformIO.

**Current version**: v1.4.0

## Architecture Overview

### Configuration System (LittleFS flash)
1. **Web Credentials** (`/web_config.json`) - HTTP Basic Auth username/password, device hostname
2. **MQTT Settings** (`/mqtt_config.json`) - Enable/disable, server, port, auth, TLS, topic prefix, HA discovery toggle, MQTT messages toggle
3. **Message Defaults** (`/defaults_config.json`) - Repeat, buzzer count, scroll delay, brightness, ASCII conversion
4. **General Settings** (`/general.config`) - Global buzzer enable, brightness override (enable + value)
5. **Clock Settings** (`/clock.config`) - NTP server, POSIX timezone, brightness, transition effect/speed/delay, randomize, resync interval, date format, date alternation (enable + seconds), custom date format, clock face, AM/PM mode
6. **Timer Settings** (`/timer.config`) - Timer vs stopwatch mode, duration, brightness, buzzer, auto-repeat, alert chirp
7. **Sleep Mode** (`/sleep_mode.config`) - Scheduled display on/off, mute-only mode, weekend override times
8. **Alarm** (`/alarm.config`) - Recurrent alarm: enable, interval, chirp name, disable-weekends, display message

**Deferred loading (ESP8266 only)**: Only defaults + general configs load before `wm.autoConnect()` to preserve heap for the WiFiManager captive portal page. All other configs load after WiFiManager completes.

### Config Objects (in `include/config.h`)
- `webConfig`: usernameWebHolder, passwordWebHolder, hostnameWebHolder
- `mqttConfig`: server, port, prefix, auth, TLS, HA discovery, MQTT messages enable
- `defaultsConfig`: REP, BUZ, DEL, BRI, ASC defaults
- `generalConfig`: buzzerEnable, brightnessOverrideEnable, brightnessOverrideValue
- `clockConfig`: enabled, ntpServer, tzString, brightness, transitionDelayMs, transitionEffect, randomizeTransition, transitionSpeed, resyncIntervalHours, dateFormat, dateAlternate, dateAlternateSeconds, customDateFormat, **clockFace**, **clockAmPm**
- `timerConfig`: enabled, mode, durationSeconds, brightness, alertBuzzer, alertChirp, autoRepeat
- `sleepModeConfig`: enabled, onTime, offTime, muteOnly, weekendEnabled, weekendOnTime, weekendOffTime
- `recurrentAlarmConfig`: enabled, interval, chirpName, disableWeekends, displayMessage

### Core Components
- **WiFiManager**: Captive portal for initial WiFi setup (AP: `RDA-MSG-XXXXXX`, Password: `wifi-setup`)
- **Web Server**: Platform-abstracted HTTP server on port 80, Basic Auth
  - ESP8266: `ESP8266WebServer` / ESP32: `WebServer`
- **PubSubClient**: MQTT client, `MQTT_MAX_PACKET_SIZE=1024`
- **MD_Parola + MD_MAX72xx**: LED matrix scrolling text (hardware SPI)
- **Fonts**: Built-in Parola font (`nullptr`), `MatrixLight8Font` (8px, PROGMEM), `MatrixLight6Font` (6px, PROGMEM)
- **LittleFS**: Flash filesystem — compatible on both platforms
- **EasyButton**: Optional FLASH button (ESP8266 only, disabled by default via `ENABLE_FLASH_BUTTON=0`)

### ESP32 FreeRTOS Task Architecture

On ESP32, CPU-intensive operations run on background FreeRTOS tasks to prevent blocking the display loop on Core 1:

- **Buzzer Task** (Core 0, 3KB stack): Plays chirp patterns via queue (1 item queue). `playChirpByName()` posts non-blocking requests; task consumes and plays.
- **HTTP Task** (Core 0, 10KB stack): Handles all HTTP requests via `serverHttp.handleClient()`. `handleHttpServer()` becomes a no-op on ESP32.
- **Crypto Fetch Task** (Core 0, 16KB stack): Triggered by binary semaphore; fetches HTTPS prices and writes to shadow buffer. Main loop swaps to live buffer when ready.
- **Weather Fetch Task** (Core 0, 12KB stack): Triggered by binary semaphore; fetches weather data and writes to WeatherShadow struct. Main loop swaps to live data when ready.

On ESP8266, these operations remain blocking (no FreeRTOS available) but are optimized to minimize display interruption.

## Feature Flags (define to disable)

| Flag | Default ESP8266 | Default ESP32 | Effect |
|------|----------------|---------------|--------|
| `DISABLE_TIMER_FEATURE` | OFF (timer enabled) | OFF | Exclude timer/stopwatch |
| `DISABLE_WEATHER_FEATURE` | **ON** | OFF | Exclude weather |
| `DISABLE_SLEEP_MODE_FEATURE` | OFF | OFF | Exclude sleep mode |
| `DISABLE_ALARM_FEATURE` | OFF | OFF | Exclude recurrent alarm |
| `DISABLE_HA_CLOCK_ADVANCED` | **ON** | OFF | Skip 7 verbose HA clock entities |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rdeangel/rda_msg_board](https://github.com/rdeangel/rda_msg_board) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
