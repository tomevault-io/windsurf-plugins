---
trigger: always_on
description: Use when you need text input from the user (naming, renaming). Shows a text field with Save/Cancel.
---

# Voice Clone Studio - Development Guidelines

## CRITICAL: Python Environment

**ALWAYS USE THE PROJECT'S VENV - NO EXCEPTIONS!**

- **NEVER use system Python or bare `python` command**
- **ALWAYS use**: `.\venv\Scripts\python.exe` (Windows) or `./venv/bin/python` (Linux/Mac)
- **Before ANY Python operation**: Use venv Python explicitly
- **Examples**:
  - `.\venv\Scripts\python.exe script.py`
  - `.\venv\Scripts\python.exe -c "import module"`
  - `python script.py` (WRONG - uses system Python)
  - `python -c "import module"` (WRONG - uses system Python)

## Code Style

**NO TYPE HINTS**

- Do NOT use type hints (no `def func(x: int) -> str`)
- Do NOT import from `typing` module
- Keep function signatures clean and simple
- Document types in docstrings if needed

## Architecture Overview

Voice Clone Studio v1.0 is fully modular. The main file (`voice_clone_studio.py`) is a ~230 line orchestrator that loads tools dynamically. All features live in `modules/`.

### Project Structure
```
voice_clone_studio.py              # Main orchestrator (~230 lines)
config.json                        # User preferences & enabled tools
modules/
├── core_components/               # OUR core app code
│   ├── __init__.py                # Core exports (modals, emotions, etc.)
│   ├── constants.py               # Central constants (models, languages, speakers)
│   ├── emotion_manager.py         # Emotion system (40+ presets)
│   ├── help_page.py               # Help content functions
│   ├── tool_base.py               # Tool/ToolConfig base classes
│   ├── audio_utils.py             # Audio processing utilities
│   ├── notification.wav           # Completion notification sound
│   │
│   ├── tools/                     # ALL UI TOOLS (tabs)
│   │   ├── __init__.py            # Tool registry, shared state builder, CSS, utilities
│   │   ├── voice_clone.py         # Voice Clone tool
│   │   ├── voice_presets.py       # Voice Presets tool
│   │   ├── conversation.py        # Conversation tool
│   │   ├── voice_design.py        # Voice Design tool
│   │   ├── sound_effects.py       # Sound Effects tool (MMAudio)
│   │   ├── prep_audio.py          # Prep Audio tool (samples + datasets)
│   │   ├── output_history.py      # Output History tool
│   │   ├── train_model.py         # Train Model tool
│   │   └── settings.py            # Settings + Help Guide tool
│   │
│   ├── ai_models/                 # AI model managers
│   │   ├── __init__.py            # get_tts_manager(), get_asr_manager()
│   │   ├── tts_manager.py         # TTS: Qwen3 Base/Custom/Design, VibeVoice, LuxTTS
│   │   ├── asr_manager.py         # ASR: Whisper, Qwen3 ASR, VibeVoice ASR
│   │   ├── foley_manager.py       # Sound Effects: MMAudio
│   │   └── model_utils.py         # Shared model utilities (device, dtype, cache, seed)
│   │
│   ├── ui_components/             # Reusable UI components
│   │   ├── __init__.py            # Component exports
│   │   ├── modals.py              # Confirmation & input modal logic
│   │   ├── confirmation_modal.*   # Confirmation modal (html/css/js)
│   │   ├── input_modal.*          # Input modal (html/css/js)
│   │   └── theme.json             # Gradio theme
│   │
│   └── gradio_filelister/         # Custom Gradio component (v0.4.0)
│
├── deepfilternet/                 # EXTERNAL: Audio denoising
├── mmaudio/                       # EXTERNAL: MMAudio sound effects
├── qwen_finetune/                 # EXTERNAL: Training logic
├── vibevoice_tts/                 # EXTERNAL: VibeVoice TTS
└── vibevoice_asr/                 # EXTERNAL: VibeVoice ASR
```

## Tool System

### How Tools Work

Each tool is a self-contained module in `modules/core_components/tools/` that:
1. Defines a `ToolConfig` with name, description, category, and default enabled state
2. Implements `create_tool(shared_state)` — creates Gradio UI inside a `gr.TabItem`
3. Implements `setup_events(components, shared_state)` — wires up event handlers
4. Returns a component dict for cross-tool communication

### Tool Registry (`tools/__init__.py`)

All tools are registered in `ALL_TOOLS`:
```python
ALL_TOOLS = {
    'voice_clone': (voice_clone, voice_clone.VoiceCloneTool.config),
    'voice_presets': (voice_presets, voice_presets.VoicePresetsTool.config),
    'conversation': (conversation, conversation.ConversationTool.config),
    'voice_design': (voice_design, voice_design.VoiceDesignTool.config),
    'prep_audio': (prep_audio, prep_audio.PrepAudioTool.config),
    'output_history': (output_history, output_history.OutputHistoryTool.config),
    'train_model': (train_model, train_model.TrainModelTool.config),
    'settings': (settings, settings.SettingsTool.config),
}
```

Users can enable/disable tools in Settings > Visible Tools, saved as `enabled_tools` in `config.json`. Settings is always visible.

### Adding a New Tool

1. **Create the module** in `modules/core_components/tools/my_tool.py`:
```python
import gradio as gr
from modules.core_components.tool_base import Tool, ToolConfig

class MyTool(Tool):
    config = ToolConfig(
        name="My Tool",
        module_name="my_tool",
        description="What this tool does",
        enabled=True,
        category="generation"
    )

    @classmethod

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FranckyB/Voice-Clone-Studio](https://github.com/FranckyB/Voice-Clone-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
