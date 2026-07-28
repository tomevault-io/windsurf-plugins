---
trigger: always_on
description: This document provides complete instructions for porting upstream changes from the pikchr diagram language to this Go port. It's designed to enable future Claude Code sessions to work autonomously on converting upstream commits.
---

# CLAUDE.md - Automated Upstream Porting Guide for Pikchr

This document provides complete instructions for porting upstream changes from the pikchr diagram language to this Go port. It's designed to enable future Claude Code sessions to work autonomously on converting upstream commits.

---

## Table of Contents

1. [Overview & Porting Philosophy](#overview--porting-philosophy)
2. [Prerequisites & Setup](#prerequisites--setup)
3. [Identifying Work to Do](#identifying-work-to-do)
4. [Step-by-Step Porting Process](#step-by-step-porting-process)
5. [Testing Changes](#testing-changes)
6. [Committing Changes](#committing-changes)
7. [Troubleshooting & When to Collaborate](#troubleshooting--when-to-collaborate)
8. [For Future Claude Sessions](#for-future-claude-sessions)

---

## Overview & Porting Philosophy

### Project Purpose

This repository (`gopikchr`) is a Go port of the pikchr diagram language parser. It tracks the upstream pikchr repository and ports changes as they occur.

### Porting Philosophy

**Principle: Similarity to C Code > Idiomatic Go**

Similar to the golemon port, this port prioritizes staying close to the original C code over making it fully idiomatic Go. The code is intentionally C-like Go code. This approach:

- Makes tracking upstream changes easier
- Allows for more mechanical conversions
- Reduces the risk of introducing bugs through over-abstraction
- Maintains structural similarity for future ports

### What Makes Pikchr Different from Golemon

**Key differences:**
1. **Source file**: We port `pikchr.y` (a Lemon grammar file), not `.c` files
2. **Both versions use parser generators**: C uses `lemonc`, Go uses `golemon`
3. **Grammar rules never change**: Only the embedded C/Go code in actions is converted
4. **Testing is critical**: Must verify C and Go produce **identical** output

### The Five Core Conversion Patterns

All pikchr conversions follow these fundamental patterns (see PIKCHR_PORTING_GUIDE.md for details):

1. **Function Signatures**: `static void func(Pik *p, ...)` → `func (p *Pik) func(...)`
2. **Pointer Access**: `pObj->field` → `pObj.field`
3. **Memory Management**: `malloc/realloc/free` → `bytes.Buffer` + GC
4. **String Handling**: `char *z, int n` → `[]byte` or `string`
5. **Macros**: `#define X` → `const X`

### Detailed Porting Reference

For detailed conversion patterns, see these comprehensive guides:

- **[PIKCHR_PORTING_GUIDE.md](PIKCHR_PORTING_GUIDE.md)** - Comprehensive 13-section guide with 100+ examples
- **[PIKCHR_PORTING_SUMMARY.md](PIKCHR_PORTING_SUMMARY.md)** - Executive summary with key insights
- **[README_PORTING_DOCS.md](README_PORTING_DOCS.md)** - Navigation guide for all documentation

---

## Prerequisites & Setup

### Required Environment

1. **This Repository** (`gopikchr`):
   ```bash
   ~/gh/p_gopikchr/gopikchr
   ```

2. **Upstream Pikchr Repository** (sibling directory):
   ```bash
   ~/gh/p_gopikchr/pikchr
   ```

   The pikchr repository must be cloned as a sibling directory:
   ```bash
   cd ~/gh/p_gopikchr/
   git clone https://github.com/drhsqlite/pikchr.git
   ```

3. **Golemon Parser Generator** (sibling directory):
   ```bash
   ~/gh/p_gopikchr/golemon/bin/golemon
   ~/gh/p_gopikchr/golemon/bin/lemonc
   ```

   The golemon repository with built binaries is required.

   **IMPORTANT**: The lemonc binary is built from `~/gh/p_gopikchr/golemon/intermediate/lemon.c`.
   This lemon.c file is periodically updated from `~/gh/p_gopikchr/pikchr/lemon.c` to get
   the latest enhancements. If you encounter lemon-related errors (e.g., `%include <file>`
   not supported), you may need to:

   ```bash
   cp ~/gh/p_gopikchr/pikchr/lemon.c ~/gh/p_gopikchr/golemon/intermediate/lemon.c
   cd ~/gh/p_gopikchr/golemon
   ./build.sh
   ```

4. **Build Tools**:
   - Go 1.x or later
   - GCC (for building C version)
   - Bash (for test scripts)

### Directory Structure

```
~/gh/p_gopikchr/
├── gopikchr/                   # This repository
│   ├── c/                      # C version (for testing)
│   │   ├── pikchr.y            # Copy of upstream pikchr.y
│   │   ├── pikchr.c            # Generated from pikchr.y using lemonc
│   │   ├── pikchr              # Compiled C binary
│   │   ├── pikchr.out          # Test output from C version
│   │   └── diff.sh             # Compares c/pikchr.y with upstream
│   ├── internal/               # Go version
│   │   ├── pikchr.y            # Go-ported grammar file
│   │   ├── pikchr.go           # Generated from pikchr.y using golemon
│   │   ├── pikchr.out          # Test output from Go version
│   │   └── dotest.sh           # Generates pikchr.go from pikchr.y
│   ├── examples/               # Test cases from upstream
│   ├── tests/                  # Additional test cases
│   ├── output/                 # Test output comparison
│   ├── dotest.sh               # Main test script
│   ├── notes.md                # Change log notes
│   └── CLAUDE.md               # This file
├── pikchr/                     # Upstream pikchr repository
│   └── pikchr.y                # Upstream source
└── golemon/                    # Parser generator

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gopikchr/gopikchr](https://github.com/gopikchr/gopikchr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
