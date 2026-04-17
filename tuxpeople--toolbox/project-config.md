---
trigger: always_on
description: Dieses Repository ist eine zentrale Sammlung von DevOps- und SysAdmin-Tools. Alle Scripts sind robust entwickelt und Enterprise-tauglich.
---

# Claude Code Anweisungen - Toolbox Repository

## 📋 Projekt-Kontext

Dieses Repository ist eine zentrale Sammlung von DevOps- und SysAdmin-Tools. Alle Scripts sind robust entwickelt und Enterprise-tauglich.

### Repository-Struktur
```
toolbox/
├── README.md                 # Haupt-Dokumentation
├── LICENSE                   # MIT-Lizenz
├── tool_name/
│   ├── tool_name            # Ausführbares Script (OHNE Dateiendung)
│   └── README.md            # Vollständige Tool-Dokumentation
└── CLAUDE.md                # Diese Datei
```

## 🛠️ Code-Standards

### Script-Organisation (ZWINGEND)
- **Eigener Ordner:** Jedes Script hat seinen eigenen Ordner
- **Bindestrich-Namen:** Tool-Namen verwenden Bindestriche (-), KEINE Unterstriche (_)
- **Keine Dateiendung:** Scripts haben KEINE Dateiendung (nicht .sh)
- **Executable:** Alle Scripts müssen ausführbar sein (`chmod +x`)
- **README.md:** Jedes Script hat ein README im gleichen Aufbau
- **Haupt-README:** Jedes Script ist im Haupt-README aufgeführt
- **Sicherheitshinweise:** Wo nötig in Haupt-README und Script-README

### Bash-Scripts (ZWINGEND)

#### Basis-Anforderungen
- **Shebang:** `#!/usr/bin/env bash` (IMMER erste Zeile)
- **Set-Optionen:** `set -euo pipefail` (IMMER nach Shebang und Kommentaren)
- **Script-Header-Kommentar:** Zweck und Verwendung kurz dokumentieren

#### Farb-Definitionen (STANDARD-PATTERN)
```bash
# Farben für Output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'  # No Color
```

#### Pflicht-Log-Funktionen (STANDARD-PATTERN)
```bash
log_info() {
    echo -e "${BLUE}[INFO]${NC} $1" >&2
}

log_success() {
    echo -e "${GREEN}[SUCCESS]${NC} $1" >&2
}

log_warn() {
    echo -e "${YELLOW}[WARN]${NC} $1" >&2
}

log_error() {
    echo -e "${RED}[ERROR]${NC} $1" >&2
}

log_verbose() {
    if [[ "$VERBOSE" == "true" ]]; then
        echo -e "${BLUE}[VERBOSE]${NC} $1" >&2
    fi
}
```

**WICHTIG:** NIEMALS `echo` direkt für Log-Ausgaben verwenden! Immer log_* Funktionen nutzen.

#### Pflicht-Funktion: show_help()
```bash
show_help() {
    cat << EOF
Tool-Name - Kurzbeschreibung

VERWENDUNG:
    $0 [OPTIONEN]

BESCHREIBUNG:
    Ausführliche Beschreibung des Tools.

OPTIONEN:
    -v, --verbose            Detaillierte Ausgabe
    -n, --dry-run            Nur anzeigen was gemacht würde
    -f, --force              Überschreibe ohne Nachfrage
    -h, --help               Diese Hilfe anzeigen

UMGEBUNGSVARIABLEN:
    VAR_NAME                 Beschreibung der Variable

BEISPIELE:
    $0                       # Standard-Verwendung
    $0 --dry-run             # Testlauf ohne Änderungen
    $0 --verbose             # Mit detaillierter Ausgabe

VORAUSSETZUNGEN:
    - tool1 muss installiert sein
    - tool2 muss verfügbar sein

HINWEISE:
    - Wichtige Hinweise zur Verwendung
    - Sicherheitsaspekte
EOF
}
```

#### Pflicht-Funktion: check_dependencies()
```bash
check_dependencies() {
    local missing_deps=()

    if ! command -v tool1 &> /dev/null; then
        missing_deps+=("tool1")
    fi

    if ! command -v tool2 &> /dev/null; then
        missing_deps+=("tool2")
    fi

    if [[ ${#missing_deps[@]} -gt 0 ]]; then
        log_error "Fehlende Abhängigkeiten: ${missing_deps[*]}"
        log_info "Installation:"
        log_info "  macOS: brew install ${missing_deps[*]}"
        log_info "  Linux: apt install ${missing_deps[*]} oder yum install ${missing_deps[*]}"
        return 1
    fi

    return 0
}
```

#### Empfohlene Funktion: parse_arguments()
Für komplexe Scripts mit vielen Optionen:
```bash
parse_arguments() {
    while [[ $# -gt 0 ]]; do
        case $1 in
            -v|--verbose)
                VERBOSE=true
                shift
                ;;
            -n|--dry-run)
                DRY_RUN=true
                shift
                ;;
            -f|--force)
                FORCE=true
                shift
                ;;
            -h|--help)
                show_help
                exit 0
                ;;
            *)
                log_error "Unbekannte Option: $1"
                show_help
                exit 1
                ;;
        esac
    done
}
```

#### Empfohlene Funktion: validate_parameters()
Für Parameter-Validierung nach dem Parsen:
```bash
validate_parameters() {
    if [[ -z "${REQUIRED_VAR:-}" ]]; then
        log_error "REQUIRED_VAR ist nicht gesetzt"
        return 1
    fi

    if [[ ! -d "${DIRECTORY:-}" ]]; then
        log_error "Verzeichnis existiert nicht: $DIRECTORY"
        return 1
    fi

    return 0
}
```

#### Standard-Optionen (Pflicht wo anwendbar)
- **`-h, --help`** - Hilfe anzeigen (PFLICHT für ALLE Scripts)
- **`-v, --verbose`** - Detaillierte Ausgabe (PFLICHT für komplexe Scripts)
- **`-n, --dry-run`** - Testlauf ohne Änderungen (PFLICHT für destruktive Operationen)
- **`-f, --force`** - Überschreibe ohne Nachfrage (OPTIONAL für destruktive Operationen)

#### Globale Variablen (Empfohlen)
```bash
# Globale Variablen am Script-Anfang
VERBOSE=false
DRY_RUN=false
FORCE=false
```

#### Statistik-Tracking (Optional für komplexe Scripts)
```bash
# Statistiken (optional)
STATS_TOTAL=0
STATS_SUCCESS=0
STATS_ERRORS=0
STATS_SKIPPED=0
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tuxpeople) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
