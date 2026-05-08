---
trigger: always_on
description: This file provides coding conventions and guidelines for TimeCapsule-Pi, a Raspberry Pi-based Time Machine backup server.
---

# AGENTS.md - Guidelines for Agentic Coding Assistants

This file provides coding conventions and guidelines for TimeCapsule-Pi, a Raspberry Pi-based Time Machine backup server.

## Project Overview

**Technology Stack:**
- **Language:** Bash scripting
- **Configuration:** INI (Samba), XML (Avahi)
- **Target OS:** Raspberry Pi OS (Debian-based)
- **Services:** Samba, Avahi
- **No:** build system, automated tests, package managers

## Commands

### No Build/Lint/Test Framework
This project uses shell scripts and configuration files. No automated build, lint, or test commands exist.

### Manual Testing Commands
```bash
# Validate Samba configuration
testparm

# Check Samba version and vfs_fruit support
smbd --version
smbd -b | grep vfs_fruit

# Check service status
systemctl status smbd nmbd avahi-daemon

# Test Samba share locally
smbclient -L localhost -U username

# List available Samba shares
smbclient -L localhost -U username% -c 2>/dev/null

# Check mount point
mountpoint -q /mnt/timecapsule
df -h | grep timecapsule

# Browse Avahi services
avahi-browse -a --terminate

# Check USB drives
lsblk
lsusb
```

### Testing Single Functions in install.sh
To test specific functions manually:
```bash
# Source the script to access functions
source ./install.sh

# Call individual functions (may require root)
check_system
detect_usb_drives
```

## Code Style Guidelines

### Shell Scripting (Bash)

**Shebang & Strict Mode:**
```bash
#!/bin/bash
set -e  # Always include - exit on error
```

**Color Output Constants:**
```bash
# Define colors at script start
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'  # No Color (reset)
```

**Function Naming:**
- Use `snake_case` for all function names
- Descriptive names: `print_header`, `check_root`, `configure_samba`
- Group related functions together

**Output Functions:**
```bash
# Standard output helpers (use these consistently)
print_header() {
    echo -e "${BLUE}...${NC}"
}

print_step() {
    echo -e "${BLUE}[STEP]${NC} $1"
}

print_info() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

print_warning() {
    echo -e "${YELLOW}[WARN]${NC} $1"
}

print_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# User confirmation prompt
confirm() {
    read -p "$(echo -e ${GREEN}[PROMPT]${NC} $1 [y/N]: )" response
    case "$response" in
        [yY][eE][sS]|[yY]) return 0 ;;
        *) return 1 ;;
    esac
}
```

**Variable Naming:**
- Constants: `UPPER_SNAKE_CASE` (e.g., `MOUNT_POINT`, `QUOTA_SIZE`)
- Local variables: `lower_snake_case` (e.g., `tm_user`, `partition`)
- Always quote variables: `"$variable"` to handle spaces

**Error Handling:**
```bash
# Always use set -e at script start
set -e

# Check for root privileges
check_root() {
    if [[ $EUID -ne 0 ]]; then
        print_error "This script must be run as root (sudo)"
        exit 1
    fi
}

# Validate commands before use
if ! command -v apt &>/dev/null; then
    print_error "apt package manager not found"
    exit 1
fi
```

**Conditional Logic:**
```bash
# Use [[ ]] for Bash conditions (more reliable than [ ])
if [[ "$os_id" != "raspbian" ]]; then
    print_warning "Not running on Raspbian"
fi

# Check array length
if [[ ${#drives[@]} -eq 0 ]]; then
    print_error "No drives found"
fi

# Numeric comparison
if [[ $mem_total -lt 512 ]]; then
    print_warning "Low memory detected"
fi
```

**Command Execution:**
```bash
# Redirect stderr to suppress noise
apt update 2>/dev/null

# Check command success with if
if mountpoint -q "$MOUNT_POINT"; then
    print_info "Drive is mounted"
fi

# Use || true to ignore expected failures
umount "$mount" 2>/dev/null || true
```

**User Input:**
```bash
# Prompt for input with validation
while true; do
    read -p "$(echo -e ${GREEN}[PROMPT]${NC} Enter choice: )" choice
    if [[ "$choice" =~ ^[0-9]+$ ]]; then
        break
    else
        print_error "Invalid input"
    fi
done
```

**Comments:**
- Use `#` for single-line comments
- Add block comments for sections
- Document non-obvious commands
- Include usage example at script start:
```bash
################################################################################
# TimeCapsule-Pi - Automated Installation Script
# Transforms Raspberry Pi into a Time Machine backup server for macOS
#
# Usage: sudo ./install.sh
################################################################################
```

### Configuration Files (INI Format - smb.conf)

**Structure:**
```ini
[section_name]
    # Comment explaining this section
    parameter = value
    parameter_with_spaces = value
```

**Conventions:**
- Use 4 spaces for indentation (not tabs)
- Comments on their own lines
- Parameter-value pairs aligned
- Group related parameters together
- Empty lines between logical sections

**Template Guidelines:**
- Use placeholders like `valid users = username` for user-specific values
- Include comprehensive comments for Time Machine options
- Keep backup of existing config before overwriting

### Configuration Files (XML Format - Avahi)

**Structure:**
```xml
<?xml version="1.0" standalone='no'?><!--*-nxml-*-->
<!DOCTYPE service-group SYSTEM "avahi-service.dtd">

<!-- Multi-line comment explaining purpose -->
<service-group>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rizal72/TimeCapsule-Pi](https://github.com/rizal72/TimeCapsule-Pi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
