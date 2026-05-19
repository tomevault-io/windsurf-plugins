---
trigger: always_on
description: Build voice apps for **all Hailo accelerators**: Whisper STT on Hailo-8/8L/10H + optional Piper TTS on CPU.
---


# Skill: Build Voice Application

Build voice apps for **all Hailo accelerators**: Whisper STT on Hailo-8/8L/10H + optional Piper TTS on CPU.

## When This Skill Is Loaded

- User wants **speech input** or **speech output** in a Hailo app
- User mentions: voice, speech, Whisper, TTS, microphone, STT, speak, listen
- User wants to add voice to an existing LLM or VLM app
- User wants speech recognition on Hailo-8 or Hailo-8L

## Hardware Compatibility

| Feature | Hailo-8/8L | Hailo-10H |
|---|---|---|
| STT (Whisper) | ✓ via `InferModel` (encoder+decoder HEFs) | ✓ via `Speech2Text` (genai API) |
| LLM on device | ✘ | ✓ via `hailo_platform.genai.LLM` |
| VLM on device | ✘ | ✓ via `Backend` (VLM chat) |
| TTS (Piper) | ✓ CPU | ✓ CPU |
| Full voice assistant | STT + CPU LLM + TTS | STT + on-device LLM + TTS |

## Reference Implementations

Study these:
- `hailo_apps/python/gen_ai_apps/voice_assistant/` — Full voice + LLM assistant (Hailo-10H)
- `hailo_apps/python/gen_ai_apps/simple_whisper_chat/` — Simple STT example (Hailo-10H)
- `hailo_apps/python/standalone_apps/speech_recognition/` — STT for **all Hailo devices** (8/8L/10H) using InferModel API:
  - `speech_recognition.py` — Main app: mic recording, audio preprocessing, transcription loop
  - `whisper_pipeline.py` — `WhisperPipeline` class: encoder+decoder inference via `InferModel`
  - `audio_utils.py` — Audio recording, mel spectrogram, file I/O
  - `postprocessing.py` — Repetition penalty and token decoding
- `hailo_apps/python/gen_ai_apps/gen_ai_utils/voice_processing/` — Voice utilities (Hailo-10H):
  - `speech_to_text.py` — `SpeechToTextProcessor` (Whisper via genai API)
  - `text_to_speech.py` — `TextToSpeechProcessor` (Piper on CPU)
  - `audio_recorder.py` — `AudioRecorder` (microphone capture)
  - `vad.py` — Voice Activity Detection
  - `interaction.py` — `VoiceInteractionManager` (high-level orchestrator)

## Build Process

### Step 1: Create App Directory

Create the app directory:

```
hailo_apps/python/<type>/<app_name>/
├── app.yaml              # App manifest (type: gen_ai)
├── run.sh                # Launch wrapper
├── __init__.py
├── <app_name>.py         # Main app
└── README.md             # Usage documentation (REQUIRED — never skip)
```

Create `app.yaml` with `type: gen_ai` and `run.sh` wrapper.
Do NOT register in `defines.py` or `resources_config.yaml`.

### Step 2: Build Main App (Hailo-10H: Voice + LLM)

```python
import signal
import threading
from contextlib import redirect_stderr
from io import StringIO

from hailo_platform import VDevice
from hailo_platform.genai import LLM

from hailo_apps.python.core.common.hailo_logger import get_logger
from hailo_apps.python.core.common.core import resolve_hef_path
from hailo_apps.python.core.common.parser import get_standalone_parser
from hailo_apps.python.core.common.defines import (
    SHARED_VDEVICE_GROUP_ID,
    HAILO10H_ARCH,
)

logger = get_logger(__name__)

from hailo_apps.python.gen_ai_apps.gen_ai_utils.voice_processing.speech_to_text import SpeechToTextProcessor
from hailo_apps.python.gen_ai_apps.gen_ai_utils.voice_processing.text_to_speech import TextToSpeechProcessor
from hailo_apps.python.gen_ai_apps.gen_ai_utils.voice_processing.interaction import VoiceInteractionManager
from hailo_apps.python.gen_ai_apps.gen_ai_utils.voice_processing.vad import add_vad_args

APP_NAME = "my_voice_app"

logger = get_logger(__name__)

APP_NAME = MY_VOICE_APP
SYSTEM_PROMPT = "You are a helpful voice assistant. Keep responses concise and natural."


def main():
    parser = get_standalone_parser()
    parser.add_argument("--no-tts", action="store_true", help="Disable TTS (text only)")
    parser.add_argument("--system-prompt", type=str, default=SYSTEM_PROMPT)
    add_vad_args(parser)
    args = parser.parse_args()

    abort_event = threading.Event()
    signal.signal(signal.SIGINT, lambda s, f: abort_event.set())

    # VDevice
    params = VDevice.create_params()
    params.group_id = SHARED_VDEVICE_GROUP_ID
    vdevice = VDevice(params)

    # STT (Whisper on Hailo)
    whisper_hef = resolve_hef_path(None, "whisper", arch=HAILO10H_ARCH)
    with redirect_stderr(StringIO()):  # Suppress ALSA noise
        stt = SpeechToTextProcessor(vdevice, str(whisper_hef))

    # LLM (on Hailo)
    llm_hef = resolve_hef_path(args.hef_path, APP_NAME, arch=HAILO10H_ARCH)
    llm = LLM(vdevice, str(llm_hef))

    # TTS (Piper on CPU)
    tts = None if args.no_tts else TextToSpeechProcessor()

    # Voice interaction manager
    vim = VoiceInteractionManager(stt, tts, abort_event)

    logger.info("Voice assistant ready. Speak into your microphone.")
    print("Voice assistant ready. Press Ctrl+C to quit.\n")

    try:
        while not abort_event.is_set():
            # Listen for speech
            user_text = vim.listen()
            if not user_text or abort_event.is_set():
                continue

            logger.info("User said: %s", user_text)
            print(f"You: {user_text}")

            # Generate response
            prompt = [
                {"role": "system", "content": [{"type": "text", "text": args.system_prompt}]},
                {"role": "user", "content": [{"type": "text", "text": user_text}]},
            ]

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hailo-ai/hailo-apps](https://github.com/hailo-ai/hailo-apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
