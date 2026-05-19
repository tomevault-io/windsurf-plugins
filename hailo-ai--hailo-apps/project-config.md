---
trigger: always_on
description: Build a complete GStreamer pipeline app for real-time video processing on Hailo-8/8L/10H.
---


# Skill: Build GStreamer Pipeline Application

Build a complete GStreamer pipeline app for real-time video processing on Hailo-8/8L/10H.

## When This Skill Is Loaded

- User wants **real-time video processing** (detection, pose, segmentation)
- User mentions: GStreamer, pipeline, stream, FPS, real-time video, tracking
- User needs a video app with **high throughput** rather than AI understanding

## Reference Implementations

The canonical pipeline app is `detection/`. Other examples: `pose_estimation/`, `instance_segmentation/`, `face_recognition/`.

**Do NOT read these source files.** This SKILL.md contains all patterns needed to build any pipeline app. The sections below cover: basic pipelines, frame overlays, custom backgrounds, pose extraction, detection data, and subclassing existing pipeline classes.

### Minimum Context for Any Pipeline App
Read this SKILL.md (full file, single read) + `common_pitfalls.md`. That's it. Build immediately.

## Build Process

### Step 1: Create App Directory

Create the app directory:

```
hailo_apps/python/<type>/<app_name>/
├── app.yaml              # App manifest (required)
├── run.sh                # Launch wrapper
├── __init__.py
├── <app_name>.py         # Main app
└── README.md             # Usage documentation (REQUIRED — never skip)
```

Create `app.yaml` with `type: pipeline` and `run.sh` wrapper.
Do NOT register in `defines.py` or `resources_config.yaml`.

### Step 2: Build Main App

```python
import gi
gi.require_version('Gst', '1.0')
from gi.repository import Gst

import hailo  # Required for detection/landmark extraction in callbacks

from hailo_apps.python.core.common.hailo_logger import get_logger
from hailo_apps.python.core.common.core import resolve_hef_path, handle_list_models_flag
from hailo_apps.python.core.common.parser import get_pipeline_parser
# If your app uses resolve_hef_path with an app name, register it in defines.py.
# Otherwise use a local string constant:
# APP_NAME = "my_pipeline_app"
from hailo_apps.python.core.gstreamer.gstreamer_app import GStreamerApp, app_callback_class
from hailo_apps.python.core.gstreamer.gstreamer_helper_pipelines import (
    SOURCE_PIPELINE,
    INFERENCE_PIPELINE,
    INFERENCE_PIPELINE_WRAPPER,
    DISPLAY_PIPELINE,
    TRACKER_PIPELINE,
    USER_CALLBACK_PIPELINE,
    QUEUE,
)
from hailo_apps.python.core.common.buffer_utils import (
    get_caps_from_pad,
    get_numpy_from_buffer,
)

logger = get_logger(__name__)

APP_NAME = "my_pipeline_app"


class UserAppCallback(app_callback_class):
    """Custom callback class for per-frame state."""
    def __init__(self):
        super().__init__()
        self.detection_count = 0


def app_callback(element, buffer, user_data):
    """Per-frame callback — runs on every GStreamer buffer."""
    # Access detections from buffer
    # user_data.detection_count += len(detections)
    return Gst.FlowReturn.OK


class MyPipelineApp(GStreamerApp):
    def __init__(self, app_callback, user_data, parser=None):
        parser = parser or get_pipeline_parser()
        handle_list_models_flag(parser, APP_NAME)
        args = parser.parse_args()
        super().__init__(args, user_data)

        self.hef_path = resolve_hef_path(args.hef_path, APP_NAME, self.arch)
        logger.info("HEF: %s", self.hef_path)

    def get_pipeline_string(self):
        return (
            SOURCE_PIPELINE(self.video_source, self.arch)
            + " ! "
            + INFERENCE_PIPELINE(
                hef_path=self.hef_path,
                batch_size=self.batch_size,
            )
            + " ! "
            + USER_CALLBACK_PIPELINE()
            + " ! "
            + DISPLAY_PIPELINE(video_sink=self.video_sink, sync=self.sync)
        )


def main():
    user_data = UserAppCallback()
    app = MyPipelineApp(app_callback, user_data)
    app.run()


if __name__ == "__main__":
    main()
```

### Step 4: Validate

```bash
python3 .hailo/scripts/validate_app.py hailo_apps/python/pipeline_apps/my_pipeline_app --smoke-test
```

## Critical Conventions

0. **USB camera input**: Always use `--input usb` for USB cameras — the framework auto-detects the correct device. **NEVER** hardcode `/dev/video0` — that is often the integrated webcam, not the USB camera. If you need a specific device, run `v4l2-ctl --list-devices` first.
1. **CLI parser**: `get_pipeline_parser()` (NOT `get_standalone_parser()`)
2. **Pipeline composition**: Use helper functions — `SOURCE_PIPELINE`, `INFERENCE_PIPELINE`, `DISPLAY_PIPELINE`
3. **Callback**: `app_callback(element, buffer, user_data)` — never call `user_data.increment()`
4. **Resolution preservation**: Use `INFERENCE_PIPELINE_WRAPPER` for full-res display
5. **Tracking**: `TRACKER_PIPELINE()` for ByteTrack
6. **Cascaded inference**: `CROPPER_PIPELINE()` for crop → second model
7. **VAAPI**: Add `QUEUE("vaapi_queue") + vaapi_convert_pipeline` for HW decode

## Common Patterns

| Pattern | Helper | Use Case |
|---|---|---|
| Basic inference | `INFERENCE_PIPELINE(hef_path=...)` | Single model |
| With tracking | `+ TRACKER_PIPELINE()` | Object tracking |
| With user callback | `+ USER_CALLBACK_PIPELINE()` | Per-frame processing |
| Cascaded | `CROPPER_PIPELINE(...)` | Face detection → recognition |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hailo-ai/hailo-apps](https://github.com/hailo-ai/hailo-apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
