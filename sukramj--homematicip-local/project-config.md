---
trigger: always_on
description: This document provides comprehensive guidance for AI assistants working with the Homematic(IP) Local for OpenCCU codebase.
---

# CLAUDE.md - AI Assistant Guide for Homematic(IP) Local Integration

This document provides comprehensive guidance for AI assistants working with the Homematic(IP) Local for OpenCCU codebase.

## Table of Contents

1. [Project Overview](#project-overview)
2. [Repository Structure](#repository-structure)
3. [Key Technologies & Dependencies](#key-technologies--dependencies)
4. [Development Environment Setup](#development-environment-setup)
5. [Code Quality & Standards](#code-quality--standards)
6. [Docstring Standards](#docstring-standards)
7. [Testing Guidelines](#testing-guidelines)
8. [Architecture & Design Patterns](#architecture--design-patterns)
9. [Common Development Tasks](#common-development-tasks)
10. [Git Workflow](#git-workflow)
11. [Implementation Policy](#implementation-policy)
12. [Tips for AI Assistants](#tips-for-ai-assistants)
13. [Quick Reference](#quick-reference)

---

## Project Overview

**Project Name:** Homematic(IP) Local for OpenCCU
**Type:** Home Assistant Custom Integration
**Version:** 2.8.0
**Primary Language:** Python 3.14+
**Domain:** `homematicip_local`

This is a production-quality Home Assistant custom integration that enables local communication with Homematic and HomematicIP smart home devices through various hubs (CCU2/3, OpenCCU, Debmatic, Homegear). It provides bi-directional communication using XML-RPC for device control and push state updates, and JSON-RPC for fetching device names and room information.

---

## Repository Structure

```
homematicip_local/
├── custom_components/homematicip_local/  # Main integration code (~7,752 lines)
│   ├── __init__.py                       # Integration setup & entry point
│   ├── config_flow.py                    # Configuration UI & validation (26.8K lines)
│   ├── control_unit.py                   # Central control logic (33.9K lines)
│   ├── services.py                       # Service registration (42.1K lines)
│   ├── services.yaml                     # Service schema definitions
│   ├── generic_entity.py                 # Base entity class (21.8K lines)
│   ├── entity_helpers.py                 # Entity creation utilities (44.4K lines)
│   ├── const.py                          # Constants & enums
│   ├── support.py                        # Helper functions
│   │
│   ├── Platform Implementations (entity types):
│   │   ├── binary_sensor.py, button.py
│   │   ├── climate.py                    # Thermostat/climate (18.5K lines)
│   │   ├── cover.py, light.py, lock.py
│   │   ├── number.py, select.py, sensor.py
│   │   ├── siren.py, switch.py, text.py
│   │   ├── update.py, valve.py
│   │
│   ├── Integration Features:
│   │   ├── device_action.py              # Device automation actions
│   │   ├── device_trigger.py             # Device automation triggers
│   │   ├── event.py                      # Custom event handling
│   │   ├── mqtt.py                       # MQTT integration
│   │   ├── logbook.py                    # Logbook integration
│   │   ├── diagnostics.py                # Diagnostic data export
│   │   ├── repairs.py                    # Issue registry integration
│   │
│   ├── Configuration:
│   │   ├── manifest.json                 # Integration metadata
│   │   ├── strings.json                  # Translatable strings (52K+ lines)
│   │   ├── translations/                 # Language files (en.json, de.json)
│   │   ├── icons.json                    # Custom icon mappings
│   │   └── quality_scale.yaml            # HACS quality score
│   │
├── tests/                                # Test suite (~1,898 lines, pytest-based)
│   ├── conftest.py                       # Test fixtures & mocks
│   ├── test_config_flow.py               # Configuration flow tests (38K+ lines)
│   ├── test_init.py                      # Integration initialization tests
│   ├── test_*.py                         # Platform-specific tests
│   │
├── .github/workflows/                    # CI/CD pipelines
│   ├── test-run.yaml                     # Main test pipeline (Python 3.14)
│   ├── pre-commit.yml                    # Code quality gate
│   ├── hacs_validate.yaml                # HACS validation
│   └── hassfest.yaml                     # HA manifest validation
│   │
├── blueprints/automation/                # HA automation blueprints
├── script/                               # Development helper scripts
│   ├── bootstrap                         # Initialize dev environment
│   ├── setup                             # Setup script
│   ├── run-in-env.sh                     # Run commands in virtualenv
│   ├── sort_class_members.py             # Class member ordering
│   └── check_translations.py             # Translation validation
│   │
├── Configuration Files:
│   ├── Makefile                          # Entry point for all dev tasks (`make help`)
│   ├── pyproject.toml                    # Project config (Python, testing, linting)
│   ├── .pre-commit-config.yaml           # Pre-commit hooks
│   ├── .yamllint                         # YAML linting rules
│   ├── codecov.yml                       # Code coverage config
│   ├── requirements_test.txt             # Testing dependencies
│   ├── Dockerfile.dev                    # Development container

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SukramJ/homematicip_local](https://github.com/SukramJ/homematicip_local) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
