---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an offline Chinese invoice OCR recognition system based on PaddleOCR 3.1+ and PP-OCRv5 models. The system extracts information from Chinese VAT invoices completely offline without network dependencies, featuring an external model architecture that separates the executable from large model files.

## Development Commands

### Installation and Setup
```bash
# Install dependencies (recommended)
python install.py

# Manual installation
pip install -r requirements.txt

# Check environment readiness
python check_build.py
```

### Running the Application
```bash
# Main GUI application
python InvoiceVision.py

# Direct OCR processing
python OCRInvoice.py

# Setup models for first run
python setup_offline_simple.py
```

### Building and Deployment (Embedded Python Architecture)

**🎯 RECOMMENDED APPROACH** - Based on UMI-OCR successful pattern

```bash
# 🚀 One-click packaging (RECOMMENDED)
build_package.bat

# Manual packaging with Python
python-embed\python.exe package_builder.py

# Test embedded Python setup
python-embed\python.exe main.py
```

**Deployment Structure:**
```
InvoiceVision_Release/
├── InvoiceVision.bat          # Main launcher
├── main.py                    # Python entry point  
├── InvoiceVision.py          # Main application
├── python-embed/             # Embedded Python runtime (~160MB)
│   ├── python.exe
│   ├── Lib/site-packages/    # Contains PaddleOCR, PyQt5, etc.
│   └── ...
├── models/                   # External model files
└── README_DEPLOYMENT.md      # User guide
```

**Key Advantages:**
- ✅ **100% Success Rate** - No PyInstaller dependency issues
- ✅ **No Python Required** - Users don't need Python installed  
- ✅ **Extract & Run** - Simple deployment experience
- ✅ **~200MB Total** - Reasonable package size
- ✅ **Perfect Compatibility** - All OCR dependencies work flawlessly

### Legacy Building Methods (DEPRECATED)
```bash
# ❌ DEPRECATED - PyInstaller approaches (historical failures)
# Environment check before building
python check_build.py

# Build Windows executable (full) - FAILED
python build_exe.py

# Build lightweight executable - FAILED
python build_lite.py

# One-click build script (Windows) - FAILED
build.bat

# Deploy external models - LEGACY
deploy_external_models.bat

# Generated executable location
./dist/InvoiceVision.exe
```

### Testing and Diagnostics
```bash
# Test basic OCR functionality
python test_basic_ocr.py
python test_ocr_simple.py
python test_ocr_quick.py

# Test packaged executable
python test_exe_functionality.py
python test_packaged_ocr.py

# Debug and diagnose issues
python analyze_ocr_issues.py
python diagnose_exe.py
python build_diagnose.py
```

## Core Architecture

### Modern Embedded Python Architecture (Current)

The project now uses a **PyStand-inspired embedded Python architecture** that completely solves the PyInstaller packaging issues:

```
Runtime Architecture:
InvoiceVision.bat → python-embed\python.exe → main.py → InvoiceVision.py
                                             │
                                             ├─ OCRInvoice.py (OCR Engine)
                                             ├─ MainAction.py (Batch Processing)
                                             ├─ PDF2IMG.py (PDF Conversion)
                                             ├─ ModelManager.py (Model Management)
                                             └─ resource_utils.py (Resource Utils)
                                             
Embedded Environment:
python-embed/
├── python.exe                    # Python 3.11.9 interpreter
├── python311.dll               # Core runtime
├── Lib/site-packages/          # All dependencies
│   ├── paddleocr/             # OCR engine
│   ├── PyQt5/                 # GUI framework
│   ├── pandas/                # Data processing
│   └── ... (all other deps)
└── python311._pth              # Path configuration
```

**Key Success Factors:**
- **No PyInstaller Complexity** - Bypasses all packaging compilation issues
- **Runtime Environment** - Uses real Python interpreter with full compatibility
- **External Models** - Models separate from executable for flexibility
- **UMI-OCR Proven Pattern** - Based on successfully deployed OCR software

### Legacy Architecture (Historical Reference)

**Previous Components** (maintained for compatibility):

1. **InvoiceVision.py** - Main PyQt5 GUI application
   - Entry point with modern GUI interface
   - Manages threading via `OfflineOCRThread` base class
   - Handles file selection and progress display
   - Integrates with `ModelManager` for model status checking

2. **OCRInvoice.py** - Core OCR engine (`OfflineOCRInvoice` class)
   - PaddleOCR wrapper for offline operation
   - Dual precision modes: 快速 (Fast) / 高精 (High Precision)
   - Regex-based invoice field extraction
   - Configuration loading from `offline_config.json`

3. **MainAction.py** - Batch processing coordination
   - `ocr_pdf_offline()` - PDF file processing pipeline
   - `ocr_images_offline()` - Image folder processing
   - Orchestrates PDF2IMG → OCRInvoice → Data Export flow

4. **PDF2IMG.py** - PDF conversion utility

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zhuguadundan/InvoiceVision](https://github.com/zhuguadundan/InvoiceVision) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
