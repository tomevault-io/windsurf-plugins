---
trigger: always_on
description: This toolkit provides local audio transcription using NVIDIA ASR models via the NeMo framework. It offers five scenarios with shared patterns for different use cases.
---

# Copilot Instructions for NVIDIA ASR Transcription Toolkit

## Project Overview

This toolkit provides local audio transcription using NVIDIA ASR models via the NeMo framework. It offers five scenarios with shared patterns for different use cases.

## Repository Structure

```
/
├── README.md                    # Main readme (only doc in root)
├── requirements.txt             # Shared dependencies
├── fix_lhotse.py               # Post-install compatibility fix
├── transcribe.py               # Legacy script (backward compatibility)
├── docs/                       # All documentation except main README
│   ├── plans/               # Implementation plans (plan_YYMMDD_HHMM.md)
│   ├── PLAN.md
│   ├── QUICKREF.md
│   ├── USAGE_EXAMPLES.md
│   └── IMPLEMENTATION_SUMMARY.md
├── utils/                      # Environment validation tools
│   ├── check_environment.py   # Validates Python, PyTorch, CUDA, dependencies
│   ├── check_models.py        # Shows model download status
│   └── README.md
├── scenario1/                  # Simple CLI transcription
│   ├── transcribe.py
│   ├── simple-transcribe.py   # Simplified transcription script
│   └── README.md
├── scenario2/                  # Interactive menu transcription
│   ├── transcribe.py
│   └── README.md
├── scenario3/                  # Multilingual transcription
│   ├── transcribe.py
│   └── README.md
├── scenario4/                  # Client-server architecture
│   ├── server/                # Python FastAPI server
│   │   ├── app.py
│   │   ├── nvidia_asr_monitor.py
│   │   ├── requirements.txt
│   │   ├── requirements-windows.txt
│   │   ├── Dockerfile
│   │   ├── setup-venv.ps1/.sh
│   │   └── README.md
│   ├── clients/
│   │   ├── console/          # C# console client
│   │   └── webapp/           # Blazor web app client
│   ├── AppHost/              # .NET Aspire orchestration
│   ├── ServiceDefaults/      # Shared Aspire service defaults
│   ├── Directory.Build.props # Shared MSBuild properties
│   ├── NvidiaTranscribe.slnx # Solution file
│   ├── test_server.py        # Server integration test
│   ├── README.md             # Only doc in scenario root
│   └── docs/                 # All other scenario docs
│       ├── QUICKREF.md
│       ├── USAGE_EXAMPLES.md
│       ├── ARCHITECTURE.md
│       ├── AZURE_DEPLOYMENT.md
│       ├── GPU_SETUP_GUIDE.md
│       ├── STRUCTURE.md
│       ├── TESTING_CHECKLIST.md
│       ├── APPINSIGHTS_QUICKSTART.md
│       ├── APPLICATION_INSIGHTS.md
│       ├── IMPLEMENTATION_COMPLETE.md
│       ├── IMPLEMENTATION_NOTES.md
│       ├── IMPLEMENTATION_SUMMARY.md
│       ├── IMPLEMENTATION_SUMMARY_SCENARIO4_ENHANCEMENTS.md
│       └── OPTION2_IMPLEMENTATION.md
├── scenario5/                  # Voice Agent (ASR + TTS + LLM)
│   ├── app.py                 # FastAPI server with WebSocket
│   ├── requirements.txt       # Scenario-specific dependencies
│   ├── README.md
│   ├── static/
│   │   └── index.html         # Browser-based voice UI
│   └── pynini_stub/           # Windows TTS compatibility
│       ├── setup.py
│       └── pynini/__init__.py
└── output/                     # Shared output directory
```

### Structure Conventions

- **Root README only**: Main `README.md` stays in repo root; all other docs go in `docs/`
- **Scenario folders**: Each scenario has its own folder with `transcribe.py` and `README.md` at the root; all other documentation goes in a `docs/` subfolder within the scenario
- **Shared resources**: `requirements.txt`, `fix_lhotse.py`, and `output/` remain in root (shared across scenarios)
- **Utils folder**: `utils/` contains environment validation scripts (`check_environment.py`, `check_models.py`)
- **Legacy support**: Root `transcribe.py` maintained for backward compatibility
- **No audio files in repo**: Audio files (`.mp3`, `.wav`, `.flac`) are gitignored; users provide their own
- **Plans**: All implementation plans are saved in `docs/plans/` with the naming convention `plan_YYMMDD_HHMM.md` (e.g., `plan_260209_1008.md` for a plan created on 2026-02-09 at 10:08). Plans capture feature proposals, architecture decisions, and implementation steps before work begins.

## Architecture & Scenarios

| Folder | Model | Use Case |
|--------|-------|----------|
| `scenario1/` | Parakeet (English) | CLI for single file transcription |
| `scenario2/` | Parakeet (English) | Interactive menu to select from local audio files |
| `scenario3/` | Canary-1B (Multilingual) | Language-specific transcription (es, en, de, fr) |
| `scenario4/` | Parakeet + Canary | Client-server architecture with REST API, .NET Aspire orchestration |
| `scenario5/` | Parakeet + FastPitch + HiFiGAN + TinyLlama | Real-time voice agent with ASR, TTS, and Smart Mode LLM |

Root `transcribe.py` is the original script (same as scenario2) - kept for backward compatibility.

## Critical Constraints

- **Python 3.10-3.12 only** - Python 3.13 is NOT supported due to NeMo/lhotse incompatibility
- **Install PyTorch with CUDA first** - Before `pip install -r requirements.txt` to ensure GPU support
- **Run `python fix_lhotse.py` after installation** - Patches lhotse for PyTorch 2.10+ compatibility

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elbruno/nvidia-transcribe](https://github.com/elbruno/nvidia-transcribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
