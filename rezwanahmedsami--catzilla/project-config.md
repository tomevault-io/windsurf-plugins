---
trigger: always_on
description: Claude: Use this guide as project context for all coding, debugging, and architectural tasks. Prioritize performance, FastAPI compatibility, and cross-platform stability. Always validate changes with benchmarks and ./scripts/run_tests.sh. Never add unnecessary dependencies.
---


<!--
Claude: Use this guide as project context for all coding, debugging, and architectural tasks. Prioritize performance, FastAPI compatibility, and cross-platform stability. Always validate changes with benchmarks and ./scripts/run_tests.sh. Never add unnecessary dependencies.
-->

# CLAUDE.md — Catzilla Project AI Agent Guide

## 🤖 Claude Code Instructions

- Assume you are a C/Python performance engineer working on Catzilla.
- Your primary goals:
  1. Maintain FastAPI-style API in Python
  2. Maximize C-side performance and safety
  3. Ensure jemalloc memory efficiency
  4. Maintain cross-platform build compatibility
- Never suggest adding unnecessary Python dependencies.
- Use only standard library + CPython extensions unless explicitly told otherwise.
- Always validate your changes with benchmarks and `./scripts/run_tests.sh`.
- Use this document as your complete reference while developing inside this project.

## Summary for AI Agents

When working on Catzilla, always remember:

1. **Performance is King**: Every change must maintain or improve performance
2. **FastAPI Compatibility**: Maintain 95% syntax compatibility
3. **Memory Efficiency**: Use jemalloc properly for 30-35% savings
4. **Cross-Platform**: Code must work on Windows, macOS, and Linux
5. **Test Everything**: Comprehensive testing is non-negotiable
6. **Document Changes**: User-facing changes need documentation
7. **Security First**: Validate inputs and handle errors properly
8. **Virtual Environments**: Always use venv for development

**Key Performance Targets:**
- Maintain 6.5-24x performance advantage over FastAPI
- Keep memory usage 30-35% lower than baseline
- Ensure zero memory leaks in C code
- Support 400,000+ RPS for static files
- Achieve sub-7ms average response times

**Critical Files to Understand:**
- `python/catzilla/app.py` - Main application class
- `src/core/router.c` - Core routing engine
- `src/core/memory.c` - Memory management
- `CMakeLists.txt` - Build configuration
- `docs/` - Comprehensive documentation

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Core Architecture](#core-architecture)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Development Guidelines](#development-guidelines)
- [Build System](#build-system)
- [Testing Framework](#testing-framework)
- [Performance Requirements](#performance-requirements)
- [Memory Management](#memory-management)
- [Feature Systems](#feature-systems)
- [Documentation Standards](#documentation-standards)
- [Code Quality Standards](#code-quality-standards)
- [File Patterns](#file-patterns)
- [Common Development Tasks](#common-development-tasks)
- [Error Handling](#error-handling)
- [Platform Compatibility](#platform-compatibility)
- [Security Considerations](#security-considerations)

---


## Project Overview

Catzilla is a Python web framework focused on:
- Extreme performance (6.5–24x faster than FastAPI)
- jemalloc-based memory efficiency (30–35% savings)
- C-accelerated core (libuv, llhttp, trie routing)
- FastAPI compatibility (95%+ syntax match)
- Zero dependencies (uses only Python standard library)

**Status:** Production ready (v0.2.0, 9/11 features complete)

**Mission:** Deliver maximum performance and developer productivity with minimal boilerplate, while maintaining FastAPI-style API and cross-platform support.

---

## Core Architecture

### Hybrid C/Python Design

```
┌─────────────────────────────────────────────────┐
│                Python Layer                     │
│  - Developer API (FastAPI-style)                │
│  - Request/Response handling                     │
│  - Business logic integration                    │
└─────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────┐
│               Python C Extension                │
│  - CPython bindings                             │
│  - Memory management bridge                     │
│  - Type conversion and safety                   │
└─────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────┐
│                 C Core Engine                   │
│  - libuv event loop                             │
│  - llhttp parser                                │
│  - Trie-based routing (O(log n))                │
│  - jemalloc memory optimization                 │
│  - Thread-safe operations                       │
└─────────────────────────────────────────────────┘
```

### Key Design Principles

1. **Performance First**: Every component optimized for maximum throughput
2. **Memory Efficiency**: jemalloc arena specialization for 30-35% savings
3. **Zero Overhead**: Minimal abstraction layers between Python and C
4. **FastAPI Compatibility**: Drop-in replacement with identical syntax
5. **Production Ready**: Enterprise-grade error handling and security
6. **Platform Agnostic**: Windows, macOS, Linux support with optimizations

---

## Technology Stack


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rezwanahmedsami/catzilla](https://github.com/rezwanahmedsami/catzilla) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
