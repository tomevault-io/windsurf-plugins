---
trigger: always_on
description: > **Status**: Draft - Comprehensive Repository Reorganization
---

# NixOS Repository Restructuring Plan

> **Status**: Draft - Comprehensive Repository Reorganization
> **Created**: 2025-11-01
> **Objective**: Optimize organization, improve accessibility, enhance maintainability

---

## Executive Summary

Este documento estabelece um plano detalhado de reestruturação completa do repositório NixOS. O objetivo é criar uma estrutura modular, intuitiva e escalável que facilite o desenvolvimento, manutenção e colaboração, enquanto mantém alta segurança e compatibilidade.

---

## Current State Analysis

### Repository Structure Overview
```
/etc/nixos/
├── flake.nix (6KB)                    # ✅ Entry point
├── laptop-offload-client.nix (11KB)   # ⚠️ Misplaced - should be in modules
├── .claude/                           # ✅ Claude Code config
├── docs/ (164KB)                      # ✅ Recently organized
├── scripts/ (48KB)                    # ✅ Good location
├── modules/ (592KB, 61 files)         # ⚠️ Needs optimization
├── hosts/ (192KB)                     # ⚠️ Single host, underutilized
├── lib/ (20KB)                        # ⚠️ Could be expanded
├── overlays/ (16KB)                   # ✅ Good structure
├── sec/ (20KB)                        # ⚠️ Confusing separation from modules/security
├── secrets/ (20KB)                    # ✅ SOPS encrypted
├── nixtrap/ (1.8MB, 39 files)         # 🔴 CRITICAL: nested git repo, legacy code
├── reports/ (76KB)                    # ⚠️ Purpose unclear
└── journal/ (32KB)                    # ⚠️ Purpose unclear
```

### Key Metrics
- **Total .nix files in modules**: 61 files
- **Total .nix files in nixtrap**: 39 files (legacy/duplicate)
- **Largest directory**: nixtrap (1.8MB) - contains nested git repository
- **Documentation**: Well organized in docs/
- **Scripts**: 4 utility scripts in scripts/

---

## Identified Issues

### 🔴 Critical Issues

1. **Nested Git Repository (nixtrap/)**
   - Contains separate git repository (git@github.com:VoidNxSEC/nixtrap.git)
   - Should be git submodule or extracted/archived
   - Creates confusion and maintenance overhead
   - 1.8MB of potentially duplicate/legacy code

2. **Security Module Fragmentation**
   - Security configs split between `modules/security/` and `sec/hardening.nix`
   - Unclear hierarchy and override semantics
   - Final override in `sec/hardening.nix` should be better documented

### ⚠️ High Priority Issues

3. **Root-Level Module (laptop-offload-client.nix)**
   - 11KB module sitting in root directory
   - Should be in `modules/services/` or `modules/system/`
   - Breaks organizational pattern

4. **Inconsistent Module Entry Points**
   - Only 5 modules have `default.nix`: shell, network/dns, services, services/users, programs
   - Should standardize: every category should have `default.nix` aggregator
   - Makes imports more verbose and error-prone

5. **Unclear Purpose Directories**
   - `reports/` (76KB) - what reports? Build reports? Audit reports?
   - `journal/` (32KB) - personal notes? System logs? Build logs?
   - Need clear documentation or removal

6. **Module Organization Gaps**
   - No clear separation between user-space and system-level configs
   - Some modules mix concerns (system/services.nix vs services/default.nix)
   - Browser configs split: `modules/browsers/` vs `modules/applications/`

### 💡 Optimization Opportunities

7. **Underutilized Library System**
   - `lib/` only contains packages.nix and shells.nix
   - Could add: helper functions, builders, common patterns
   - Missing: validation functions, type definitions, reusable builders

8. **Single Host Configuration**
   - Only `hosts/kernelcore/` exists
   - Structure prepared for multi-host but underutilized
   - Could better separate host-specific vs shared configs

9. **Documentation Distribution**
   - Most docs in `docs/` but `.claude/CLAUDE.md` contains critical info
   - Should have clear README.md in root
   - Module-level documentation missing

10. **No Testing Infrastructure**
    - No `tests/` directory
    - No integration test framework
    - Checks in flake.nix are minimal (fmt, iso, vm, docker-app)

---

## Restructuring Plan

### Phase 1: Critical Path (Week 1)

#### 1.1 Address Nested Git Repository

**Options for nixtrap/:**
- **Option A (Recommended)**: Convert to git submodule
  ```bash
  git rm -rf nixtrap/
  git submodule add git@github.com:VoidNxSEC/nixtrap.git nixtrap
  ```
- **Option B**: Archive and extract useful modules
  ```bash
  mv nixtrap/ archive/nixtrap-$(date +%Y%m%d)/
  # Extract useful modules to modules/
  ```
- **Option C**: Keep as separate repository reference in docs

**Decision Matrix**:
- Use **Option A** if nixtrap is actively developed separately
- Use **Option B** if nixtrap is legacy and contains useful modules
- Use **Option C** if nixtrap is pure reference material

#### 1.2 Relocate Root-Level Modules

```bash
# Move laptop-offload-client.nix to proper location
mv laptop-offload-client.nix modules/services/laptop-offload-client.nix

# Update flake.nix import
# FROM: ./laptop-offload-client.nix
# TO:   ./modules/services/laptop-offload-client.nix
```

#### 1.3 Clarify Security Module Hierarchy

**Current Structure**:
```
modules/security/*.nix  (imported first)
sec/hardening.nix      (imported last - final override)
```

**Proposed Structure**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VoidNxSEC/nixos](https://github.com/VoidNxSEC/nixos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
