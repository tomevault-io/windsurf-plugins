---
trigger: always_on
description: **You are working ON ContextKit, not WITH ContextKit!**
---

# ContextKit Repository Development Guide

## 🚨 CRITICAL: Meta-Development Context

**You are working ON ContextKit, not WITH ContextKit!**

This repository contains **TEMPLATES and INSTALLATION SCRIPTS** that other developers will use. The files here are **NOT meant to be executed directly** in this repository - they are **source templates** that get copied to user systems during installation.

## 🧠 Understanding the Meta-Architecture

ContextKit is a "template distribution system" - this repository contains:
1. **Templates** - Files that get copied and customized during installation  
2. **Installation scripts** - Logic that copies templates to user systems
3. **Documentation** - Explains how the system works

**Key Mental Model**: Think of this repository as a "template factory" that produces customized development environments for other projects.

---

## 📂 Complete Repository Structure & Purpose

```
ContextKit/
├── 📄 install.sh                    # GLOBAL INSTALLER (users run: curl | sh)
├── 📖 README.md                     # Public documentation  
├── 📖 CLAUDE.md                     # This file - development guidance for AI assistants
├── 📖 LICENSE                       # MIT license
├── 📄 CHANGELOG.md                  # Version tracking and migration support
└── 🎯 Templates/                     # TEMPLATE DISTRIBUTION CENTER
    ├── Guidelines/                  # → GLOBAL CODING STANDARDS (copied by install.sh)
    │   ├── Swift.md                 # Swift patterns
    │   └── SwiftUI.md               # SwiftUI patterns
    ├── Commands/                    # → CLAUDE CODE COMMANDS (get copied during /ctxk:proj:init)
    │   ├── proj/                    # Project management commands
    │   │   ├── init.md              # Project initialization
    │   │   ├── init-workspace.md    # Workspace configuration
    │   │   └── migrate.md           # Version updates
    │   ├── plan/                    # Feature planning workflow
    │   │   ├── 1-spec.md            # Business requirements
    │   │   ├── 2-research-tech.md   # Technical research and architecture
    │   │   └── 3-steps.md           # Implementation breakdown
    │   ├── impl/                    # Development workflow
    │   │   ├── start-working.md     # Context-aware development start
    │   │   ├── commit-changes.md    # Smart commit message generation
    │   │   ├── release-app.md       # iOS/macOS App Store releases
    │   │   └── release-package.md   # Swift Package releases
    │   └── bckl/                    # Backlog management
    │       ├── add-idea.md          # Capture new ideas with evaluation
    │       ├── add-bug.md           # Bug report with impact assessment
    │       ├── prioritize-ideas.md  # Organize ideas backlog
    │       ├── prioritize-bugs.md   # Triage bugs backlog
    │       ├── remove-idea.md       # Remove completed ideas from backlog
    │       └── remove-bug.md        # Remove fixed bugs from backlog
    ├── Scripts/                     # → ALL SCRIPTS (hooks & standalone, get copied during /ctxk:proj:init)
    │   ├── AutoFormat.sh           # Auto-format edited Swift files (PostToolUse hook)
    │   ├── VersionStatus.sh        # Version checking and status display (SessionStart hook)
    │   └── CustomStatusline.sh     # Complete statusline script with 5h-usage tracking and colored progress bars
    ├── Agents/                      # → AI ASSISTANTS (get copied during /ctxk:proj:init)
    │   ├── build-project.md         # [READY] Execute builds with constitutional compliance checking
    │   ├── check-accessibility.md   # [INCOMPLETE] Accessibility compliance validation (VoiceOver, contrast, etc.)
    │   ├── check-localization.md    # [INCOMPLETE] Localization readiness audit (String Catalog, cultural adaptation)
    │   ├── check-error-handling.md  # [INCOMPLETE] ErrorKit pattern validation and typed throws
    │   ├── check-modern-code.md     # [INCOMPLETE] API modernization (Date.now, Duration, async/await)
    │   ├── check-code-debt.md       # [INCOMPLETE] Technical debt cleanup and code consolidation
    │   ├── run-test-suite.md        # [READY] Execute complete test suite with build validation and structured failure reporting
    │   └── run-specific-test.md     # [READY] Execute specific test with build validation and focused failure analysis
    ├── Features/                    # → FEATURE TEMPLATES (used by /ctxk:plan:1-spec, /ctxk:plan:quick, etc.)
    │   ├── Spec.md                  # Business requirements and user stories (spec-kit methodology)
    │   ├── Tech.md                  # Technical architecture and constitutional compliance
    │   ├── Steps.md                 # Implementation task breakdown with parallel markers [P]
    │   └── Quick.md                 # Quick task planning template with interactive validation
    ├── Contexts/                    # → CONTEXT TEMPLATES (used by /ctxk:proj:init and /ctxk:proj:init-workspace)
    │   ├── Project.md               # Project-level Context.md with ContextKit configuration
    │   └── Workspace.md             # Workspace-level Context.md with client/company overrides
    ├── Backlog/                     # → BACKLOG TEMPLATES (used by /ctxk:bckl:add-idea and /ctxk:bckl:add-bug)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FlineDev/ContextKit](https://github.com/FlineDev/ContextKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
