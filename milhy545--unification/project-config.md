---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Project Overview

**Unification** is a multi-server infrastructure automation system born from real operational pain: **150+ SSH configuration failures** across a 3-server ecosystem. The core insight: 80% of problems stemmed from context limitations (repeating the same mistakes). Solution: intelligent automation with self-discovery, idempotency, and comprehensive testing of "impossible scenarios."

**3-PC Ecosystem Architecture**:
- **HAS** (192.168.0.58, 100.90.137.86): Home Automation Server - "Heart" (orchestration, ZEN Coordinator port 8020)
- **LLMS** (192.168.0.41, 100.68.65.121): LLM Server - "Brain" (Ollama on port 11434)
- **Workstation** (192.168.0.10, Aspire-PC): Development machine - "Hands"
- **Network**: Local 192.168.0.x/24 + Tailscale VPN 100.x.x.x, SSH port 2222 (non-standard)

---

## Development Commands

### Setup & Installation
```bash
# Install package in development mode
pip install -e .

# Install dependencies
pip install -r requirements.txt

# Install dev dependencies
pip install flake8 pytest
```

### Running the System
```bash
# Interactive wizard (main entry point)
python master_wizard.py

# Dry-run mode (safe exploration without making changes)
python wizards/workstation_setup.py --dry-run
```

### Testing
```bash
# Run all tests
python -m unittest discover tests

# Run specific test module
python -m unittest tests.impossible_scenarios.test_ssh_chaos

# Run single test case
python -m unittest tests.impossible_scenarios.test_ssh_chaos.TestSSHChaos.test_port_confusion_nightmare
```

### Linting
```bash
# Run flake8 (CI uses this)
flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
flake8 . --count --max-complexity=10 --max-line-length=127 --statistics
```

### CI/CD
GitHub Actions runs on push/PR to main:
- Python 3.9, 3.10, 3.11 matrix
- Installs package with `pip install -e .`
- Lints with flake8
- Runs pytest

---

## Code Architecture

### Core Design Philosophy

**"Test the impossible, automate the repeatable, validate everything"**

1. **Impossible**: Test scenarios that actually occurred (port confusion, permission hell, network partitions, context loops)
2. **Repeatable**: Workstation setup with dry-run and idempotency
3. **Validate**: Comprehensive checks before/after every operation

### Component Interaction Flow

```
master_wizard.py (orchestrator)
    ↓
    ├─→ tools/system_detector.py      # Discover hardware, OS, capabilities
    ├─→ tools/dependency_resolver.py  # Cross-distro package management
    ├─→ tools/network_scanner.py      # Find ecosystem servers
    └─→ tools/config_validator.py     # Validate SSH, tmux, shell configs
         ↓
    wizards/workstation_setup.py (only implemented wizard)
         ↓
    [gather → analyze → plan → confirm → execute → validate]
```

### Key Modules

#### `master_wizard.py` - Entry Point
- Bilingual menu (Czech/English via MenuOption dataclass)
- 5 setup scenarios + 3 operations (ecosystem integration, post-reinstall recovery, health check)
- Orchestrates all tools and wizards
- Dry-run support throughout

#### `tools/system_detector.py` - System Intelligence
```python
SystemInfo dataclass:
  - OS detection via /etc/os-release
  - CPU thermal capabilities (Q9550-specific detection)
  - Virtualization detection (hypervisor, Docker, Podman)
  - Security audit (sudo, SSH, firewall, SELinux/AppArmor)
  - Health checks (disk <10%, memory >90%, CPU load)
  - JSON export
```

#### `tools/dependency_resolver.py` - Package Management
```python
PackageManager enum: APT, YUM, DNF, Pacman, APK, Brew, PIP, Conda, Snap
Package dataclass: OS-specific name mapping
  - 23+ common packages (git, python3, docker, nodejs, etc.)
  - Conflict detection (yum vs dnf)
  - Installation order optimization
  - Time/space estimates via InstallationPlan
```

#### `tools/network_scanner.py` - Network Discovery
```python
ServerInfo dataclass with service identification
  - Parallel scanning (ThreadPoolExecutor, max_workers=50/20)
  - Known ecosystem patterns (60% port match threshold):
    * orchestration: [2222, 8123, 3000, 9000, 8020]
    * llm_server: [2222, 8080, 11434]
  - SSH detection (ports 22 and 2222)
  - NetworkTopology JSON export
```

#### `tools/config_validator.py` - Health Validation
```python
ValidationResult dataclass (ok/warning/error)
  - SSH config validation (file, permissions, server status)
  - Tmux setup validation
  - Shell config validation (.shell_common sourcing)
  - Network validation (localhost, DNS, interfaces)
  - EcosystemValidation with recommendations
```

#### `wizards/workstation_setup.py` - Implementation Example
```python
WorkstationConfig dataclass
Phases:
  1. gather requirements
  2. analyze system
  3. create plan
  4. confirm with user
  5. execute with idempotency
  6. validate completion

Package groups:
  - REQUIRED: git, python3, tmux, curl, htop, ssh
  - DEVELOPMENT: docker, nodejs, gcc, make
  - AI_TOOLS: ollama-llm (conditional)

Features:
  - Dry-run support (--dry-run)
  - Error recovery (SSH config revert on failure)
  - Idempotent checks (don't reinstall if already configured)
```

### Architectural Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/milhy545) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
