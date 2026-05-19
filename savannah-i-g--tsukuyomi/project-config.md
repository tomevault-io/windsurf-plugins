---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. I have provided this so that you can use and edit this yourself -
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. I have provided this so that you can use and edit this yourself - 
### Do Not Load This File if you are using in another LLM

## System Architecture

TSUKUYOMI is an intelligent modular framework for structured analysis and processing, designed for professional intelligence operations. The architecture follows a component-based design with three main layers:

### Core Components
- **tsukuyomi_core.tsukuyomi**: Central orchestration system managing module loading, execution, and workflow transitions
- **key.activationkey**: Framework initialization instructions defining file interpretation rules and activation sequences
- **personality/amaterasu_personalitycore.tsukuyomi**: Communication and interaction management system with stakeholder optimization
- **modules/**: Directory containing specialized operational modules (*.tsukuyomi files)

### File Format
All framework components use `.tsukuyomi` extension containing JSON-like structures. These files are:
- Text files with JSON syntax and custom schema
- Modular prompt engineering components
- Self-contained but interoperable modules
- Processed sequentially during activation

### Framework Operation
The system follows a specific initialization sequence:
1. Load and parse tsukuyomi_core.tsukuyomi
2. Extract configuration parameters (module_path, personality_path)
3. Load AMATERASU personality core
4. Initialize core functions and context management
5. Present greeting and await user input or module selection

## Module System

### Module Types
- **Core Intelligence Modules**: Data recognition, correlation analysis, discipline alignment
- **Economic Analysis (E-Series)**: E1-E4 modules for economic vulnerability and trade analysis
- **Strategic Analysis (S-Series)**: S1-S4 modules for geopolitical scenario modeling and capability assessment
- **Infrastructure & Security**: Critical infrastructure vulnerability and dependency mapping
- **Specialized Operations**: Flight data analysis, OSINT collection, comprehensive reporting
- **Source Quality & Bias Analysis**: Source correlation matrix, bias detection, AI hallucination detection
- **Session Management**: Session export/import modules for analytical continuity

### Module Architecture
Each module contains:
- Standard schema with id, version, title, type, dependencies
- Input requirements and output schema definitions
- Execution sequences with specific actions and conditions
- Error handling and security protocols
- IC-compliant analytical frameworks

## Communication Protocols

The framework uses standardized message prefixes:
- `//AMATERASU:` - System communications
- `//TSUKUYOMI:` - Framework operations  
- `//RESULT:` - Analytical findings
- `//QUERY:` - Clarification requests
- `//ANOMALY:` - Unusual patterns
- `//CRITICAL:` - Urgent information
- `//CLASSIFICATION:` - Security markings
- `//SELECTION:` - Menu option selections

## Framework Activation

### Splash Screen Interface

TSUKUYOMI includes an interactive splash screen that displays upon initialization (when `enable_splash_screen: true` in core configuration). The splash screen presents a compact menu with checkbox-style options (☐) for different operational modes:

1. Standard Analysis Mode - General analytical operations
2. Intelligence Operations Mode - Professional intelligence ops with security
3. Economic Analysis Suite - Economic vulnerability & trade analysis
4. Strategic Analysis Suite - Scenario modeling & capability assessment  
5. Infrastructure & Security Analysis - Critical infrastructure assessment
6. Specialized Operations - OSINT, flight data, custom reporting
6a. Source Quality & Bias Analysis - Source correlation matrix & bias detection
7. Module Browser - View all available modules
8. System Configuration - Configure security & operational parameters
9. Session Management - Export/import session snapshots for continuity

### Activation Methods

**Interactive Menu (with splash screen enabled):**
```
Initialise Amaterasu
```
System responds with menu and: `//AMATERASU: Welcome to TSUKUYOMI. Please select from the options above or provide your analytical requirements directly.`

**Direct Activation:**
```
Initialize TSUKUYOMI for intelligence operations, classification level [UNCLASSIFIED/CONFIDENTIAL/SECRET]
```

**Menu Selection:**
Users can select by number (1-9) or provide analysis requests directly, bypassing the menu.

**Return to Menu:**
Users can return to the main menu at any time by typing:
```
menu
```
System responds: `//AMATERASU: Returning to main menu. Please select from the options below.`

### Configuration Options
- `enable_splash_screen: true/false` - Toggle splash screen display
- `compact_display: true/false` - Use compact formatting
- `menu_return_command: true` - Enable menu return capability
- Menu can be disabled for direct operational use

## Security Considerations

The framework implements comprehensive security protocols:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [savannah-i-g/TSUKUYOMI](https://github.com/savannah-i-g/TSUKUYOMI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
