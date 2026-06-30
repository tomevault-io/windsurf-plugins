---
trigger: always_on
description: This workspace contains **PiKISS** (Pi Keeping It Simple, Stupid!) - a collection of Bash scripts with a menu system designed to automate software installation and configuration on Raspberry Pi devices. The project aims to simplify the process of setting up applications, emulators, development tools, and system configurations.
---


# PiKISS Workspace Context

## Project Overview

This workspace contains **PiKISS** (Pi Keeping It Simple, Stupid!) - a collection of Bash scripts with a menu system designed to automate software installation and configuration on Raspberry Pi devices. The project aims to simplify the process of setting up applications, emulators, development tools, and system configurations.

## Required Structure

```bash
#!/bin/bash
# Description : Descriptive name
# Author      : Jose Cerrejon Gonzalez (ulysess@gmail_dot._com)
# Version     : 1.0.0 (DD/MM/YY)
# Compatible  : Raspberry Pi 4, 5

. ./scripts/helper.sh || . ../helper.sh || wget -q 'https://github.com/jmcerrejon/PiKISS/raw/master/scripts/helper.sh'
clear
check_board || { echo "Missing file helper.sh..." && exit 1; }

# Required variables
readonly INSTALL_DIR="$HOME/apps"
readonly PACKAGES=(package1 package2)
readonly BINARY_URL="$PIKISS_REMOTE_SHARE_DIR_URL/app-name.tar.gz"
```

## Required Variables

-   `readonly INSTALL_DIR` (use $HOME/games or $HOME/apps)
-   `readonly PACKAGES=()`
-   `readonly BINARY_URL` (domain: misapuntesde.com/rpi_share/)

## Standard Functions

-   Include `runme()`, `uninstall()`, `install()`
-   Use `install_packages_if_missing`
-   Call `generate_icon` for GUI applications
-   End with `exit_message`

## User Messages

-   Clear description before installation
-   Show final path and controls/keys
-   Confirm destructive actions with [y/N]

## Instructions for AI

1. **ALWAYS verify helper.sh at start**
2. **USE readonly for all URLs and constants**
3. **INCLUDE informative message with install_script_message**
4. **GENERATE desktop icon if GUI application**
5. **HANDLE both precompiled binaries and compilation**
6. **CHECK architecture with is_userspace_64_bits for specific binaries**
7. **USE dialog for user interaction**
8. **USE functions from helper.sh for common tasks**
9. **FOLLOW the structure and patterns established in existing scripts**

## Key Components

-   **Main script**: [`piKiss.sh`](piKiss.sh) - Entry point with dialog-based menu system
-   **Helper functions**: [`scripts/helper.sh`](scripts/helper.sh) - Common utilities and functions
-   **Script categories**:
    -   `scripts/games/` - Game installations (GTA, VVVVVV, Captain S, etc.)
    -   `scripts/devs/` - Development tools (VSCode, Qt5, Docker, etc.)
    -   `scripts/emus/` - Emulators (PSP, Commodore 64, etc.)
    -   `scripts/config/` - System configurations (Vulkan, networking)
    -   `scripts/server/` - Server applications (Jenkins, Git server)
    -   `scripts/others/` - Miscellaneous tools (Alacritty, Wine, etc.)

## Target Platform

-   Primary: Raspberry Pi 4 and 5
-   OS: Raspberry Pi OS (Bullseye/Bookworm), 64-bit support
-   Also compatible with other ARM-based systems

## Common Patterns

-   Scripts use `dialog` for interactive menus
-   Functions like `install_packages_if_missing`, `download_and_extract`, `make_with_all_cores`
-   Consistent structure: install from binary vs compile from source options
-   Icon generation and desktop entry creation
-   Automatic dependency management

## Practical Examples

### 1. Basic Script Structure

```bash
#!/bin/bash
#
# Description : Example script
# Author      : Jose Cerrejon Gonzalez (ulysess@gmail_dot._com)
# Version     : 1.0.0 (DD/MM/YY)
#
. ./scripts/helper.sh || . ../helper.sh || wget -q 'https://github.com/jmcerrejon/PiKISS/raw/master/scripts/helper.sh'
clear
check_board || { echo "Missing file helper.sh..." && exit 1; }

readonly INSTALL_DIR="$HOME/apps"
readonly PACKAGES=(package1 package2)
readonly BINARY_URL="$PIKISS_REMOTE_SHARE_DIR_URL/app.tar.gz"

install() {
    install_packages_if_missing "${PACKAGES[@]}"
    download_and_extract "$BINARY_URL" "$INSTALL_DIR"
    generate_icon
}

uninstall() {
    read -p "Do you want to uninstall? (y/N) " response
    if [[ $response =~ [Yy] ]]; then
        delete_dir "$INSTALL_DIR/app"
    fi
}

runme() {
    install
    exit_message
}

# Entry point
install_script_message
runme
```

### 2. Handling Different Architectures

```bash
if is_userspace_64_bits; then
    readonly BINARY_URL="$PIKISS_REMOTE_SHARE_DIR_URL/app-arm64.tar.gz"
else
    readonly BINARY_URL="$PIKISS_REMOTE_SHARE_DIR_URL/app-armhf.tar.gz"
fi
```

### 3. Desktop Icon Generation (GUI)

```bash
generate_icon() {
    if [ ! -d "$HOME/.local/share/applications" ]; then
        mkdir -p "$HOME/.local/share/applications"
    fi

    cat << EOF > "$HOME/.local/share/applications/app.desktop"
[Desktop Entry]
Name=App Name
Comment=App Description
Exec=$INSTALL_DIR/app/app
Icon=$INSTALL_DIR/app/icon.png
Terminal=false
Type=Application
Categories=Game;
EOF
}
```

### 4. Build from Source

```bash
compile() {
    install_packages_if_missing "${PACKAGES_DEV[@]}"
    cd "$HOME/source" || exit 1
    git clone "$SOURCE_URL" && cd "$_" || exit 1
    ./configure
    make_with_all_cores
    make_install_compiled_app
    exit_message
}
```

## Author


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmcerrejon/PiKISS](https://github.com/jmcerrejon/PiKISS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
