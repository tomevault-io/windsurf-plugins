---
trigger: always_on
description: 1. [Überblick](#überblick)
---

# Eugen – Intelligenter Gaming & 3D-Druck Twitch-Bot
## Komplettes Setup & Implementierungs-Guide für Windows

---

## 📋 Inhaltsverzeichnis

1. [Überblick](#überblick)
2. [Systemanforderungen](#systemanforderungen)
3. [Installation & Setup](#installation--setup)
4. [Konfigurationsassistent](#konfigurationsassistent)
5. [GUI Dashboard](#gui-dashboard)
6. [Architektur & Implementierung](#architektur--implementierung)
7. [API-Integration & Debugging](#api-integration--debugging)
8. [Fehlerbehandlung](#fehlerbehandlung)
9. [Projektstruktur](#projektstruktur)

---

## Überblick

**Eugen** ist ein intelligenter Twitch-Chat-Agent für:
- **Gaming**: World of Warcraft, Elden Ring, Gamedev
- **3D-Druck**: Prusa i3, Bambu, Creality
- **Tech**: Python, Linux, Home Automation

### Kernfeatures

| Feature | Beschreibung |
|---------|-------------|
| **Name Recognition** | Erkennt automatisch wenn angesprochen (@Eugen, Eugen:, etc.) |
| **Persistent Memory** | Speichert Chat-History pro User (max 25 Nachrichten) |
| **Context-Aware** | Antwortet basierend auf vorherigem Gesprächsverlauf |
| **Perplexity Integration** | Nutzt Perplexity Sonar für Echtzeit-Web-Suche |
| **Live Monitoring** | GUI zeigt alle API-Calls, Responses, Fehler in Echtzeit |
| **Windows-Native** | Vollständige Windows-Unterstützung, keine Linux-Tools nötig |

---

## Systemanforderungen

### Minimal
- Windows 10/11
- Python 3.9+
- 100 MB Festplatte
- Internetverbindung

### Empfohlen
- Python 3.11+
- 500 MB Festplatte (für Chat-History & Logs)
- Stabiles Netzwerk (für IRC & API)

---

## Installation & Setup

### Schritt 1: Python installieren

1. Gehe zu [python.org](https://www.python.org/downloads/)
2. Lade **Python 3.11+** herunter
3. **WICHTIG**: Häkchen setzen bei "Add Python to PATH"
4. Installieren

**Verifizieren:**
```powershell
python --version
pip --version
```

### Schritt 2: Projekt initialisieren

```powershell
# Neuer Ordner für Eugen
mkdir C:\Users\YourUsername\eugen
cd C:\Users\YourUsername\eugen

# Virtual Environment erstellen
python -m venv venv

# Aktivieren (Windows PowerShell)
.\venv\Scripts\Activate.ps1

# Falls Fehler bei Ausführungsrichtlinie:
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
# Dann nochmal: .\venv\Scripts\Activate.ps1
```

### Schritt 3: Dependencies installieren

```powershell
pip install -r requirements.txt
```

**requirements.txt:**
```
perplexity-python-client==1.0.0
irc==20.1.0
python-dotenv==1.0.0
PySimpleGUI==4.60.0
requests==2.31.0
```

---

## Konfigurationsassistent

### Setup-Flow (Automatisch beim ersten Start)

Beim ersten Start wird ein **interaktives Setup-Fenster** angezeigt:

```
╔════════════════════════════════════════════════════════════════╗
║               EUGEN KONFIGURATIONSASSISTENT                    ║
╠════════════════════════════════════════════════════════════════╣
║                                                                ║
║  1️⃣  TWITCH KONFIGURATION                                     ║
║  ───────────────────────────────────────────────────────────  ║
║  Bot-Name:           [___________________________]             ║
║  OAuth Token:        [***hidden***] [🔑 Anleitung]            ║
║  Channel:            [___________________________]             ║
║                                                                ║
║  2️⃣  PERPLEXITY KONFIGURATION                                ║
║  ───────────────────────────────────────────────────────────  ║
║  API Key:            [***hidden***] [🔑 Anleitung]            ║
║  Model:              [sonar-pro ▼]                            ║
║  Max Tokens:         [450]                                     ║
║                                                                ║
║  3️⃣  BOT-VERHALTEN                                            ║
║  ───────────────────────────────────────────────────────────  ║
║  ☑ Context Memory aktivieren                                 ║
║  ☑ Name Recognition aktivieren                               ║
║  ☑ Debug-Mode (zeigt API-Calls)                             ║
║                                                                ║
║  [Speichern & Starten]  [Abbrechen]                           ║
║                                                                ║
╚════════════════════════════════════════════════════════════════╝
```

### API-Keys beschaffen

#### Twitch OAuth Token

1. Gehe zu [twitchtokengenerator.com](https://twitchtokengenerator.com/) **ODER** manuell:
   - [Twitch Developer Console](https://dev.twitch.tv/console/apps)
   - Neue Anwendung erstellen
   - OAuth-Authentifizierung
   - Scopes: `chat:read`, `chat:edit`
   - Token generieren

2. **Token Format:** `oauth:abcd1234efgh5678...`
3. **Speichern in Setup-Fenster**

**⚠️ WICHTIG:** Token niemals ins Repository committen!

#### Perplexity API Key

1. Gehe zu [perplexity.ai/api](https://www.perplexity.ai/api)
2. Registrieren / Anmelden
3. API Keys → Neuen Key generieren
4. **Format:** `pplx-abcd1234efgh5678...`
5. **Speichern in Setup-Fenster**

---

## GUI Dashboard

### Live Monitoring Interface

Während der Bot läuft, zeigt das Dashboard in Echtzeit:

```
╔════════════════════════════════════════════════════════════════════╗
║                    EUGEN BOT - LIVE DASHBOARD                     ║

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kenearos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
