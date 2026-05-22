---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## MANDATORY WORKFLOW - NO EXCEPTIONS

### BEFORE ANY WORK (BLOCKING REQUIREMENTS)
**🛑 STOP: Complete ALL steps below before any implementation**

- [ ] **MANDATORY**: Enter plan mode first (if not already)
- [ ] **MANDATORY**: Create TodoWrite with task tracking
- [ ] **MANDATORY**: Write detailed plan to `.claude/tasks/TASK_NAME.md`
  - Problem statement and root cause analysis
  - Detailed implementation plan with reasoning
  - Tasks broken down into specific steps
  - Success criteria and test plan
- [ ] **MANDATORY**: If task requires external knowledge, use Task tool for research
- [ ] **MANDATORY**: Keep plan focused (MVP approach, don't over-plan)
- [ ] **MANDATORY**: Use ExitPlanMode tool to request user approval
- [ ] **🛑 BLOCKING**: Do NOT proceed until user explicitly approves the plan

### WHILE IMPLEMENTING (REQUIRED UPDATES)
**These are NOT optional - MUST be done throughout work:**

- [ ] **MANDATORY**: Update TodoWrite tool with progress as you work
- [ ] **MANDATORY**: Update task document in `.claude/tasks/` with implementation details
- [ ] **MANDATORY**: Document any deviations from original plan with reasoning
- [ ] **MANDATORY**: Append detailed descriptions of each change for engineer handoff

### AFTER COMPLETING WORK (COMPLETION REQUIREMENTS)
**🛑 STOP: Complete ALL steps before considering task done**

- [ ] **MANDATORY**: Update task document with final results and test outcomes
- [ ] **MANDATORY**: Update TodoWrite to mark all tasks as completed
- [ ] **MANDATORY**: Update task status to ✅ COMPLETED in `.claude/tasks/TASK_NAME.md`
- [ ] **MANDATORY**: Verify all changes are documented for easy handoff to other engineers
- [ ] **MANDATORY**: Include file paths, line numbers, and rationale for all changes

### WORKFLOW ENFORCEMENT
**If you find yourself implementing without following this workflow:**
1. **IMMEDIATELY STOP** what you are doing
2. **CREATE** the missing task document
3. **DOCUMENT** what you've already done
4. **ASK FOR APPROVAL** before continuing
5. **FOLLOW** the complete workflow going forward

**No exceptions. No shortcuts. This ensures consistency and trackability.**

### SELF-MONITORING CHECKLIST

Before proceeding with ANY task, verify:

**Pre-Work Validation** (Before any implementation):
- [ ] I am in plan mode
- [ ] I have created a TodoWrite with this task
- [ ] I have written a plan to `.claude/tasks/TASK_NAME.md` using the template
- [ ] I have used ExitPlanMode to request approval
- [ ] The user has explicitly approved my plan
- [ ] I understand exactly what needs to be done

**Mid-Work Validation** (Check every 15 minutes):
- [ ] I am updating TodoWrite with my progress
- [ ] I am documenting changes in the task file as I make them
- [ ] I am following the approved plan (or documenting deviations)
- [ ] I can explain each change I'm making and why

**Post-Work Validation** (Before marking complete):
- [ ] All changes are documented with file paths and line numbers
- [ ] All TodoWrite tasks are marked completed
- [ ] Task status is updated to ✅ COMPLETED
- [ ] Another engineer could understand and continue my work
- [ ] I have tested the changes work as expected

**Emergency Stop Conditions**:
If ANY of these occur, IMMEDIATELY stop and follow the enforcement workflow:
- 🚨 I realize I'm implementing without a plan
- 🚨 I'm modifying code without documenting it
- 🚨 I can't remember what I'm supposed to be doing
- 🚨 The user hasn't approved my plan yet

## Project Overview

WisprFlow Lite is a dual-version voice-to-text transcription application that works system-wide. It uses OpenAI's Whisper API for transcription and provides push-to-talk functionality. The project offers both a Python CLI version for developers and an Electron GUI app for end users.

## Dual-Version Architecture

### Python CLI Version (`python-cli/`)
- **Status**: ✅ Ready to use
- **Target**: Developers and power users
- **Multiple stable versions available**:
  - `voice_transcriber_openai.py` - Original OpenAI implementation
  - `voice_transcriber_fireworks.py` - Fireworks AI integration with filler word removal (66% faster)
  - `voice_transcriber_fireworks_no_filler.py` - **RECOMMENDED** - Fireworks AI without text processing (66% faster, maximum stability)
- **Key components**:
  - Audio recording with PyAudio
  - Fireworks AI Whisper Turbo API integration 
  - Performance-optimized logging for maximum speed
  - Global keyboard listener using pynput
  - Text injection via pyautogui
  - Robust error handling and resource cleanup
  - Configuration via `.env` files

### Electron GUI App (`electron-app/`)
- **Status**: 🚧 In development
- **Target**: All users wanting a native Mac app
- **Architecture**: Electron wrapper around Python backend
- **Key files**:
  - `src/main/main.js`: Electron main process
  - `src/renderer/`: Frontend interface
  - `src/python/`: Python backend integration
  - IPC communication between Electron and Python processes

### Configuration Structure
- **Python CLI**: `.env` files in `python-cli/` directory
- **Electron app**: Configuration managed through GUI with backend `.env` generation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tommyyau/wispr-flow-lite](https://github.com/tommyyau/wispr-flow-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
