---
trigger: always_on
description: **Project:** Privacy-focused Linux port for HY300 Android projector
---

# GitHub Copilot Instructions for HY300 Linux Porting

**Project:** Privacy-focused Linux port for HY300 Android projector  
**Hardware:** Allwinner H713 (sun50iw12p1) SoC  
**Approach:** Hardware-first, evidence-based development with zero bricking tolerance  
**Last Updated:** November 4, 2025

---

## 🎯 Project Architecture at a Glance

### The "Big Picture"
This is a **hardware porting project**, not a typical software build. We're replacing Android factory firmware with a privacy-focused Armbian custom ROM. The unique challenge: we can't brick the device, so every modification must be validated on hardware first.

**Key Architectural Decision: Hardware-First + Research Integration**
- **Hardware-First:** Live system analysis from two HY300 devices is ground truth
- **Research Integration:** 100+ analysis documents (`research/`) provide patterns and context
- **Atomic Validation:** Each phase locks down one layer before moving up the stack
- **A/B Testing:** Device A (development), Device B (control) prevents catastrophic failures
- **Recovery Always Enabled:** UART console must work before any risky bootloader changes

### Directory Structure & Purpose
```
hy300-linux-porting/
├── phases/                          # EXECUTION - Phase-based development (I-VIII)
│   ├── phase1-hardware-baseline/   # Phase I: Document current state (ACTIVE)
│   ├── phase2-uart-access/         # Phase II: UART serial console setup
│   ├── phase3-uboot-replacement/   # Phase III: Bootloader swap (HIGH RISK)
│   └── [phase4-8]                  # Kernel, drivers, ROM, security, validation
│
├── tasks/                           # TASK TRACKING - Active work items
│   ├── pending/                    # Not yet started
│   ├── in-progress/                # Currently being worked (ONE ONLY)
│   └── completed/                  # Finished and validated
│
├── ai/                              # AI INFRASTRUCTURE
│   ├── contexts/                   # Context docs (phase integration, research workflow)
│   └── tools/                       # task-manager, context-manager, git-manager
│
├── research/                        # SOFTWARE ANALYSIS ARCHIVE (100+ docs)
│   ├── docs/                       # Firmware analysis, driver reverse engineering
│   ├── firmware/                   # Extracted Android ROM components
│   ├── drivers/                    # Kernel module references from research
│   └── tools/                      # Analysis utilities (firmware parsers, etc.)
│
├── hardware-access/                 # HARDWARE PROCEDURES
│   ├── root-access-guide.md        # ADB/SSH procedures for Device A+B
│   ├── uart-setup.md               # Serial console connection
│   └── hardware-dump-procedures.md # Complete backup procedures
│
└── agents/                          # AGENT GUIDELINES
    └── AGENT_GUIDELINES.md         # Detailed agent protocols and delegation
```

### Critical Files for Agent Productivity
- **`README.md`** (487 lines) - Project status, hardware access, phase overview
- **`PROJECT_ROADMAP.md`** (1096 lines) - All 8 phases, timelines, success criteria
- **`phases/README.md`** (309 lines) - Phase navigation and critical docs index
- **`phases/RECOVERY_TEMPLATE.md`** (719 lines) - Recovery procedures for every scenario
- **`phases/phase2-uart-access/UART_BOOTLOADER_SAFETY_PROTOCOL.md`** (557 lines) - Safe SRAM testing
- **`phases/research-validation/RESEARCH_MAPPING.md`** (576 lines) - Hardware findings validated

---

## 🛡️ Hardware Architecture & Safety Model

### Hardware Stack (Bottom-Up)
```
BOOTLOADER (U-Boot)
  ↑ verified via UART before replacing
  
KERNEL (Mainline Linux)  
  ↑ flashed to eMMC after bootloader proven
  
DRIVERS (H713-specific modules)
  ↑ reverse-engineered from Android or written from datasheet
  
ARMBIAN ROM (Userspace)
  ↑ built with custom privacy patches
```

### A/B Device Testing Strategy
- **Device A:** Primary development, all dangerous changes tested here first
- **Device B:** Factory firmware control device, baseline for comparison
- **Safety Rule:** Never modify Device B until proven safe on Device A
- **Recovery Path:** Any Device A failure → restore from complete backup via UART FEL mode

### The "Zero Bricking" Architecture
1. **Before Phase III (U-Boot swap):** UART must be fully tested and working
2. **SRAM Testing First:** All bootloader changes tested via SRAM before flashing eMMC
3. **Factory Kernel Preserved:** Android kernel remains on device until Phase IV ready
4. **Complete Backups Mandatory:** Every phase has verified backup procedures
5. **Decision Gates:** Major phases require documented validation before proceeding

---

## 🚀 Critical Workflows & Developer Patterns

### Workflow #1: Starting Daily Work
```bash
# ALWAYS FIRST - Check for incomplete work
ai/tools/task-manager find-inprogress

# If task found → continue it
# If nothing found → get next priority
ai/tools/task-manager next

# Start your task (moves to in-progress/)
ai/tools/task-manager start <task_id>
```

**Why This Matters:** The project spans weeks. Tasks might be incomplete from previous sessions. Always resume before starting new work.

### Workflow #2: Adding New Hardware Feature
1. **Check research:** `phases/research-validation/RESEARCH_MAPPING.md`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [srgneisner/hy300-linux-porting](https://github.com/srgneisner/hy300-linux-porting) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
