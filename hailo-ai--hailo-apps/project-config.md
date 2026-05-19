---
trigger: always_on
description: Build a complete Vision-Language Model application that uses the Hailo-10H VLM for image understanding.
---


# Skill: Build VLM Application

Build a complete Vision-Language Model application that uses the Hailo-10H VLM for image understanding.

## When This Skill Is Loaded

- User wants to build an app that **looks at camera images and answers questions**
- User needs **visual scene understanding** (describe, count, detect, analyze)
- User wants a variant of the VLM Chat app with different behavior
- User mentions: VLM, vision, image understanding, camera monitoring, scene analysis

## Reference Implementation

Study `hailo_apps/python/gen_ai_apps/vlm_chat/` — the canonical VLM app:
- `vlm_chat.py` — State machine app with camera loop
- `backend.py` — Multiprocessing VLM inference backend (REUSE this, don't copy)

## Build Process

### Step 1: Create App Directory

Create the app directory:

```
hailo_apps/python/<type>/<app_name>/
├── app.yaml              # App manifest (required)
├── run.sh                # Launch wrapper (sets PYTHONPATH)
├── __init__.py           # Empty
├── <app_name>.py         # Main app class + entry point
├── event_tracker.py      # Optional: event classification (for monitoring apps)
└── README.md             # Usage documentation (REQUIRED — never skip)
```

**app.yaml** — required manifest:
```yaml
name: <app_name>
title: My VLM App
description: One-line description
author: AI Agent (auto-generated)
date: "YYYY-MM-DD"
type: gen_ai
hailo_arch: hailo10h
model: Qwen2-VL-2B-Instruct
tags: [vlm, monitoring]
status: draft
```

**run.sh** — launch wrapper:
```bash
#!/bin/bash
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
REPO_ROOT="$(cd "$SCRIPT_DIR/../../.." && pwd)"
PYTHONPATH="$REPO_ROOT" python3 "$SCRIPT_DIR/<app_name>.py" "$@"
```

**NOTE**: Do NOT register in `defines.py` or `resources_config.yaml`.
Community apps are run via `run.sh`. Registration happens during promotion

### Step 2: Build the App

The main app file follows this structure:
```python
import os
import sys
import cv2
import signal
import time
from typing import Optional

os.environ["QT_QPA_PLATFORM"] = 'xcb'

from hailo_apps.python.gen_ai_apps.vlm_chat.backend import Backend
from hailo_apps.python.core.common.core import (
    get_standalone_parser, resolve_hef_path, handle_list_models_flag
)
from hailo_apps.python.core.common.defines import (
    HAILO10H_ARCH, USB_CAMERA
)
from hailo_apps.python.core.common.camera_utils import get_usb_video_devices
from hailo_apps.python.core.common.hailo_logger import get_logger

logger = get_logger(__name__)

APP_NAME = "my_vlm_app"

SYSTEM_PROMPT = "Your system prompt here..."
MONITOR_PROMPT = "Your per-frame VLM question here..."

class MyVLMApp:
    def __init__(self, camera, camera_type, args):
        self.camera = camera
        self.camera_type = camera_type
        self.running = True
        self.backend = None
        signal.signal(signal.SIGINT, self.signal_handler)
        # Initialize Backend, EventTracker, etc.

    def signal_handler(self, sig, frame):
        self.running = False

    def run(self):
        # Main loop: capture frame, display, analyze periodically
        pass

    def cleanup(self):
        if self.backend:
            self.backend.close()
        cv2.destroyAllWindows()

def main():
    parser = get_standalone_parser()
    # IMPORTANT: Add ALL custom args BEFORE handle_list_models_flag
    # so they appear in --help output
    parser.add_argument("--interval", type=int, default=15, help="Seconds between analyses")
    handle_list_models_flag(parser, APP_NAME)
    args = parser.parse_args()
    hef_path = resolve_hef_path(args.hef_path, app_name=APP_NAME, arch=HAILO10H_ARCH)
    # Camera setup, app.run()

if __name__ == "__main__":
    main()
```

### Step 4: Validate

Run the automated validation script (includes static checks + runtime smoke tests):
```bash
python3 .hailo/scripts/validate_app.py hailo_apps/python/gen_ai_apps/<app_name> --smoke-test
```

### Step 5: Write README

Include: description, requirements, usage CLI, architecture, customization notes.

## Key Customization Points

| What to Change | Where |
|---|---|
| System prompt | `SYSTEM_PROMPT` constant |
| Per-frame VLM question | `MONITOR_PROMPT` constant |
| Image preprocessing | `Backend.convert_resize_image()` |
| Inference parameters | `MAX_TOKENS`, `TEMPERATURE` |
| Event classification | `EventTracker.classify_response()` |
| Display overlay | OpenCV `cv2.putText()` in main loop |

## Display & Output Best Practices

### Window Size
The VLM crops images to 336×336 but this is too small for a display window.
Always resize to at least 640×640 for readability:
```python
DISPLAY_SIZE = (640, 640)
display = cv2.resize(frame, DISPLAY_SIZE, interpolation=cv2.INTER_LINEAR)
```

### Text Wrapping
VLM responses can be long (100+ chars). Always wrap overlay text to fit the window:
```python
@staticmethod
def _wrap_text(text: str, max_chars: int = 70) -> list[str]:
    words = text.split()
    lines, current = [], ""
    for word in words:
        if current and len(current) + 1 + len(word) > max_chars:
            lines.append(current)
            current = word
        else:
            current = f"{current} {word}".strip() if current else word
    if current:
        lines.append(current)
    return lines or [""]
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hailo-ai/hailo-apps](https://github.com/hailo-ai/hailo-apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
