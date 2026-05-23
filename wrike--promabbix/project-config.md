---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Promabbix is a tool that connects Prometheus to Zabbix monitoring. It converts Prometheus alert rules into Zabbix templates, allowing you to monitor Kubernetes/Prometheus metrics within Zabbix while maintaining your existing monitoring infrastructure.

## Key Architecture Components

### CLI Interface (`src/promabbix/promabbix.py`)
- **Click-based CLI**: Modern command-line interface with subcommands
- **Main Commands**: `generateTemplate` (implemented), `syncTemplate` (planned), `syncPseudoHosts` (planned)
- **Help System**: Comprehensive help text and command documentation

### Generate Template Command (`src/promabbix/cli/generate_template.py`)
- **GenerateTemplateCommand**: Main command class that handles template generation workflow
- **Dependency Injection**: Accepts DataLoader, DataSaver, and ConfigValidator instances for testability
- **Error Handling**: Rich console output with colored validation messages and proper exit codes
- **STDIN/STDOUT Support**: Pipeline-friendly design for integration with other tools

### Core Modules (`src/promabbix/core/`)
- **template.py**: Jinja2-based template rendering engine with custom filters and globals from Ansible
  - `Render` class handles template processing with extensive Jinja2 customization
  - Integrates Ansible filters for data manipulation (combine, regex operations, JSON/YAML handling)
- **fs_utils.py**: File system operations for loading/saving YAML/JSON data
  - `DataLoader`: Handles input from files or STDIN with automatic YAML/JSON detection
  - `DataSaver`: Unified save_to_file method with format detection (.json/.yaml/.yml)
  - Supports both structured data and string content with smart format handling
- **validation.py**: Comprehensive configuration validation system
  - `ConfigValidator`: Schema validation with enhanced error reporting (supports both YAML and JSON schemas)
  - `CrossReferenceValidator`: Validates consistency between config sections
  - `SchemaValidator`: JSON Schema validation with detailed error messages and suggestions
  - Wiki documentation validation for alerts (optional but recommended)
  - Uses `unified.yaml` schema by default (backward compatible with JSON schemas)
- **migration.py**: Legacy format migration utilities
  - Converts legacy three-file format to unified YAML format
  - Supports detection and automatic migration of legacy configurations
- **data_utils.py**: Utility functions for data validation (JSON checking)

### Template System
- Main template: `prometheus_alert_rules_to_zbx_template.j2` - Complex Jinja2 template that converts Prometheus recording rules and alerting rules into Zabbix template JSON format
- Template processes both recording_rules (Prometheus queries) and alerting_rules (Zabbix triggers)
- Handles label mapping, macro substitution, and Zabbix-specific formatting

## Development Commands

### CLI Usage
```bash
# Show main help
PYTHONPATH=src python -m promabbix.promabbix --help

# Generate template help
PYTHONPATH=src python -m promabbix.promabbix generateTemplate --help

# Validate configuration only
PYTHONPATH=src python -m promabbix.promabbix generateTemplate examples/minikube-alert-config.yaml --validate-only

# Generate template to file
PYTHONPATH=src python -m promabbix.promabbix generateTemplate examples/minikube-alert-config.yaml -o output.json

# Generate template to STDOUT
PYTHONPATH=src python -m promabbix.promabbix generateTemplate examples/minikube-alert-config.yaml

# Process STDIN input
cat examples/minikube-alert-config.yaml | PYTHONPATH=src python -m promabbix.promabbix generateTemplate -
```

### Testing
```bash
# Run all tests
python -m pytest tests/ -v

# Run tests with coverage
python -m pytest tests/ -v --cov=src/promabbix --cov-report=term-missing

# Run with coverage requirement (80% minimum)
python -m pytest tests/ -v --cov-fail-under=80 --cov=src/promabbix --cov-report=term-missing --cov-report=xml

# Run using the test runner script (recommended)
python run_tests.py

# Run specific test file
python -m pytest tests/test_cli_generate_template.py -v
```

### Docker Build and Usage
```bash
# Build Docker image
docker buildx build -t promabbix:local .

# Run with help
docker run promabbix:local

# Process file with mounted directory
docker run --mount type=bind,src=$(pwd)/examples/,dst=/mnt promabbix:local /mnt/minikube-alert-config.yaml -o /mnt/output.json

# Use STDIN/STDOUT for pipeline processing
cat examples/minikube-alert-config.yaml | docker run -i promabbix:local - -o -
```

### Local Development
```bash
# Install dependencies
pip install -r requirements.txt
pip install -r requirements-dev.txt

# Run locally
python3 src/promabbix/promabbix.py examples/minikube-alert-config.yaml -o output.json
```

### Code Quality (CI Commands)
```bash
# Run complete CI quality check locally
source .venv/bin/activate

# Run tests with coverage (80% minimum required)
python -m pytest tests/ -v --cov-fail-under=80 --cov=src/promabbix --cov-report=term-missing --cov-report=xml

# Run flake8 linting
flake8 src/ --count --max-complexity=10 --max-line-length=127 --statistics


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wrike/promabbix](https://github.com/wrike/promabbix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
