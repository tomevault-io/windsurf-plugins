---
trigger: always_on
description: **Integration Reminders:**
---

# Before You Start

**Integration Reminders:**
- Use Flask's built-in `request.form` and `request.files` for form data; the server no longer requires the `cgi` module.
- The Flask app renders each page with Jinja templates and standard HTML forms. No global JavaScript handler is required.
- When adding a new feature, create a route in `move-webserver.py`, a template in `templates_jinja/`, and add a corresponding link in `templates_jinja/base.html` with a matching `active_tab` value.

# Extending Move

This guide explains how to add new features to the Move webserver.

The project uses a single Flask-based server implemented in `move-webserver.py`.

Move follows a modular structure, with each feature comprising three components:

- **Core Logic**: Business logic (e.g., audio processing) in `core/`
- **Web Handler**: HTTP request handling in `handlers/`
- **HTML Template**: User interface in `templates/`

## Project Structure

```
extending-move/
├── move-webserver.py      # Flask web server
├── core/                  # Core functionality implementations
│   ├── slice_handler.py         # Sample slicing and kit creation
│   ├── refresh_handler.py       # Library refresh via D-Bus
│   ├── reverse_handler.py       # WAV file reversal
│   ├── time_stretch_handler.py      # Time-stretching (with WSOLA/Phase-vocoder & repitch options)
│   ├── restore_handler.py       # Move Set restoration
│   ├── drum_rack_inspector_handler.py   # Preset inspection and modification
│   ├── synth_preset_inspector_handler.py  # Legacy macro management
│   ├── synth_param_editor_handler.py  # Drift parameter and macro editing
│   ├── set_management_handler.py    # MIDI set generation and management
│   └── midi_pattern_generator.py    # Custom MIDI pattern creation utilities
├── handlers/              # Web request handlers
│   ├── base_handler.py                    # Base handler with shared functionality
│   ├── slice_handler_class.py             # Slice kit creation interface
│   ├── refresh_handler_class.py           # Library refresh interface
│   ├── reverse_handler_class.py           # WAV reversal interface
│   ├── restore_handler_class.py           # Move Set restoration interface
│   ├── drum_rack_inspector_handler_class.py  # Preset inspection interface
│   ├── synth_preset_inspector_handler_class.py  # Legacy macro management interface
│   ├── synth_param_editor_handler_class.py  # Drift preset editor interface
│   ├── set_management_handler_class.py     # MIDI set generation and upload interface
│   └── file_placer_handler_class.py       # File upload and placement
├── templates_jinja/       # Jinja templates used by the Flask app
│   ├── base.html                # Shared layout and navigation
│   ├── index.html               # Landing page
│   ├── chord.html               # Chord generation interface
│   ├── slice.html               # Waveform slicing interface
│   ├── reverse.html             # WAV reversal interface
│   ├── restore.html             # Move Set restoration
│   ├── drum_rack_inspector.html # Drum rack inspection
│   ├── synth_params.html        # Drift preset editor (parameters & macros)
│   └── midi_upload.html         # MIDI file upload and set creation
├── static/               # JavaScript and CSS assets
├── examples/              # Example files for testing and development
│   ├── Track Presets/          # Sample presets organized by instrument type
│   │   ├── Drift/              # Drift instrument presets
│   │   ├── Wavetable/          # Wavetable instrument presets
│   │   ├── drumRack/           # Drum rack presets
│   │   └── melodicSampler/     # Melodic sampler presets
│   ├── Midi/                   # Example MIDI files for testing
│   ├── Sets/                   # Example Ableton Live sets and templates
│   ├── pattern_examples.py     # MIDI pattern generation examples
│   └── test scripts            # Various test scripts
└── utility-scripts/       # Installation and management scripts
    ├── install-on-move.sh     # Initial setup script
    ├── update-on-move.sh      # Update deployment script
    └── restart-webserver.sh   # Server management script
├── tests/                 # pytest unit tests for the Flask app
```

Each feature should follow this structure precisely:
- Core logic: `core/feature_name_handler.py`
- Handler class: `handlers/feature_name_handler_class.py`
- UI template: `templates_jinja/feature_name.html`

## Core Components

### 1. Core Handlers
Core handlers implement the main functionality of each feature. They should:
- Focus on core logic without web-specific code
- Handle file operations and data processing
- Return structured results with success/failure status
- Include comprehensive error handling

Example core handler structure:
```python
def process_feature(param1, param2):
    """
    Main function implementing feature logic.
    
    Args:
        param1: Description
        param2: Description
    
    Returns:
        dict with keys:
        - success: bool indicating success/failure
        - message: Status or error message
        - Additional result data as needed
    """
    try:
        # Implementation
        return {
            'success': True,
            'message': 'Operation completed',
            'data': result_data
        }
    except Exception as e:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [charlesvestal/extending-move](https://github.com/charlesvestal/extending-move) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
