---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Session Protocol

**At the start of each session, always ask:**
> What would you like to work on today?

Wait for explicit response before taking action. This ensures alignment on priorities and scope.

**For complex implementation tasks**, use EnterPlanMode to explore the codebase, design a solution, and get approval before coding.

**For tracking progress** on multi-step work, use TaskCreate/TaskUpdate to manage task status.

**For persistent information**, save to the memory system at `C:\Users\aabou\.claude\projects\C--Users-aabou-PycharmProjects-OsirisDevworks-sc-profile-editor\memory\`. This carries context across sessions.

---

## Development Workflow

### 1. Planning Phase
When tackling non-trivial changes:
- **Enter planning mode** (`EnterPlanMode` tool) to explore the codebase and design the solution
- Walk through the plan step-by-step
- **Iterate with the user** until the full plan is approved
- Identify affected files, testing strategy, and documentation updates needed

### 2. Implementation Phase
- Implement changes step-by-step, checking in between logical blocks
- Keep changes focused and minimal (avoid scope creep)
- Follow existing code patterns and conventions (see Code Patterns section)
- Update relevant documentation alongside code changes
- Run manual testing to verify behavior

### 3. Testing Phase
- Test both **Standard view** (3-column) and **Detailed view** (6-column)
- Verify exports work in all formats (CSV, PDF, Word, Graphics)
- Test label editing and persistence
- Check version displays correctly in window title
- Test all tabs: Controls Table, Device View, Config, About
- **For device detection work:** Test with actual hardware if available

### 4. Documentation Phase
- Update `docs/CHANGELOG.md` under `[Unreleased]` section with changes
- Update `README.md` if user-facing functionality changed
- Update `docs/DEVELOPMENT.md` if build/setup changed
- Add comments only where logic isn't self-evident

### 5. Commit & Review Phase
- Draft commit message summarizing the change
- **Show commit message for review** before committing
- Create clear, descriptive commits (avoid "Fix stuff")
- Use conventional commit format when appropriate

### 6. Version Management
No version increment needed until release is ready. When releasing:
- **Patch** (0.8.2 → 0.8.3): Bug fixes only
- **Minor** (0.8.2 → 0.9.0): New features or device support
- **Major** (0.8.2 → 1.0.0): Breaking changes or major rewrites

---

## Project Overview

**SC Profile Editor** is a desktop application for Star Citizen players to edit and export their control profiles in human-readable formats. It generates visual diagrams of controller layouts and exports bindings to PDF, Word, CSV, and graphical formats.

- **Platform:** Windows only (PyInstaller executable)
- **Language:** Python 3.12+
- **GUI Framework:** PyQt6
- **Current Version:** 0.8.2 (released 2026-02-07)
- **Status:** Active development

---

## Project Architecture

### High-Level Data Flow

```
Load Profile (XML)
    ↓
Parse XML → Detect Devices → Generate Labels → Display Table
    ↓
Apply Device Mappings & Template Selection
    ↓
Edit Labels → Save Overrides → Update Device View & Table
    ↓
Export → Apply Filters & View Mode → Generate Output (CSV/PDF/Word/PNG)
```

### Core Components

#### 1. **Action Registry System** (`src/registry/action_registry.py`)
- Loads complete action database from `UNBIND_ALL.xml` (1,085 actions across ~50 actionmaps)
- Loaded at startup, indexed by action name and organized by category
- Used for: dropdown suggestions, action validation, label generation
- The BLANK profile (`example-profiles/BLANK.xml`) extends this with all 1,085 actions

#### 2. **Parser & Label Generation** (`src/parser/`)
- `xml_parser.py`: Parses Star Citizen profile XML files, extracts bindings and actionmaps
- `label_generator.py`: Creates human-readable labels for actions using three-tier system (see below)

#### 3. **Label Override System** (`src/utils/label_overrides.py`)
- **Tier 1 (Custom)**: User-edited labels stored in `label_overrides_custom.json` (AppData)
- **Tier 2 (Global)**: Default labels in `label_overrides.json` (bundled with app)
- **Tier 3 (Auto-generated)**: Fallback from action name if no override exists
- Labels update in **real-time** across table and device graphics

#### 4. **Device Graphics System** (`src/graphics/`, `src/gui/qtpdf_device_widget.py`)
- PDF-based device templates with interactive form fields
- QtPdf (native Qt renderer) for lightweight PDF viewing (no Chromium)
- PyMuPDF (fitz) for PDF manipulation and form field access
- Pattern-based device matching in `visual-templates/template_registry.json`
- Supports **composite devices** (stick + module) via `device_splitter.py`
- Clickable PDF fields open RemapDialog for action assignment

#### 5. **Input Detection System** (`src/utils/input_detector.py`)
- **Joystick**: Buttons, axes, POV/hat switches via `pygame`
- **Keyboard**: Keys with modifier support (Ctrl, Alt, Shift) via `pynput`
- **Mouse**: Button clicks (5-button mice) via `pynput`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Osiris-DevWorks/sc-profile-editor](https://github.com/Osiris-DevWorks/sc-profile-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
