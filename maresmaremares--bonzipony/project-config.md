---
trigger: always_on
description: AI-powered Windows desktop pet: voice interaction, autonomous behavior, screen monitoring, desktop control, multi-pony system. Built on PyQt5, Whisper STT, ElevenLabs/PVT TTS, and multiple LLM backends.
---

# CLAUDE.md — bonziPONY Codebase Guide

AI-powered Windows desktop pet: voice interaction, autonomous behavior, screen monitoring, desktop control, multi-pony system. Built on PyQt5, Whisper STT, ElevenLabs/PVT TTS, and multiple LLM backends.

## Architecture Overview

```
main.py (bootstrap + wiring)
  │
  ├─ Activation Thread ─── wake word / PTT / double-click detection
  │       │
  │       └─ Pipeline Thread ─── IDLE → ACK → LISTEN → THINK → SPEAK → convo loop
  │               │
  │               ├─ stt/transcriber.py      Whisper STT
  │               ├─ llm/ providers          LLM call (chat/generate_once)
  │               ├─ llm/response_parser.py  extract tags from LLM output
  │               └─ core/tts_queue.py       enqueue speech (priority-ordered)
  │
  ├─ Agent Loop Thread ─── 1-3s ticks, autonomous behavior
  │       │
  │       ├─ core/screen_monitor.py    free window title polling (no LLM)
  │       ├─ directives.json           persistent tasks with urgency 1-10
  │       ├─ core/routines.py          scheduled recurring actions
  │       └─ core/event_timeline.py    shared event log (thread-safe)
  │
  ├─ TTSQueue Consumer Thread ─── serialized audio playback
  │       │
  │       └─ tts/ engines              ElevenLabs or OpenAI-compatible
  │
  └─ Main Thread (Qt) ─── GUI event loop
          │
          ├─ desktop_pet/pet_window.py     sprite animation (~60fps)
          ├─ desktop_pet/speech_bubble.py  comic-style response display
          ├─ desktop_pet/heard_text.py     STT transcription overlay
          └─ desktop_pet/context_menu.py   right-click settings UI
```

## Thread Safety Rules

| Component | Thread(s) | Sync mechanism |
|-----------|-----------|----------------|
| Pipeline | Activation thread spawns it | Isolated; one conversation at a time |
| Agent Loop | Own daemon thread | `_conversation_active` flag silences it during user interaction |
| TTSQueue | Any thread enqueues; consumer thread plays | `PriorityQueue` + `_seq_lock` |
| EventTimeline | Pipeline + Agent both write | `threading.Lock` on all reads/writes |
| TTS engine | Pipeline + TTSQueue both call speak() | **`_tts_lock` in main.py** wraps speak() |
| Qt GUI updates | Must happen on main thread | `PetController` uses Qt signals with `QueuedConnection` |

**Critical**: Never call Qt widget methods from background threads. Always go through PetController signals.

## File Ownership Map

### core/ — Brain and coordination
| File | Owns | Key class |
|------|------|-----------|
| `pipeline.py` | Conversation state machine (wake→listen→think→speak) | `Pipeline` |
| `agent_loop.py` | Autonomous behavior, directives, enforcement, AFK mischief | `AgentLoop` |
| `tts_queue.py` | Priority-ordered multi-pony audio serialization | `TTSQueue` |
| `pony_manager.py` | Multi-pony lifecycle, voice routing, group chat scheduling | `PonyManager` |
| `pony_instance.py` | Per-pony state bundle (GUI + LLM + sprites + config) | `PonyInstance` |
| `group_conversation.py` | Inter-pony turn-taking conversations | `GroupConversation` |
| `routines.py` | Persistent scheduled actions (wake/sleep/daily/weekly/interval) | `RoutineManager` |
| `event_timeline.py` | Shared event log bridging Pipeline and AgentLoop | `EventTimeline` |
| `screen_monitor.py` | Win32 window title polling (free, no API calls) | `ScreenMonitor` |
| `config_loader.py` | YAML config → typed dataclasses | `AppConfig` and sub-configs |
| `character_registry.py` | Scans Ponies/ dirs, maps slugs ↔ display names | `scan_ponies()` |
| `memory.py` | Session summaries persisted across restarts | `save_summary()`, `load_recent()` |
| `user_profile.py` | Extracted user facts (name, interests, events) | `load_profile()`, `update_from_conversation()` |
| `diary.py` | Per-character in-character journal | `write_entry()`, `read_recent()` |
| `monitor_utils.py` | Win32 multi-monitor bounds via ctypes | `get_monitor_rect_for_point()` |
| `audio_utils.py` | Audio device enumeration helpers | `list_pyaudio_devices()` |
| `updater.py` | Git-based self-update from GitHub | `check_for_updates()` |

### llm/ — LLM abstraction
| File | Owns |
|------|------|
| `base.py` | Abstract `LLMProvider` interface: `chat()`, `generate_once()`, `describe_image()` |
| `factory.py` | Provider routing: Anthropic, OpenAI, OpenRouter, DeepSeek, Groq, Ollama, local servers |
| `anthropic_provider.py` | Claude SDK with retry logic and vision support |
| `openai_provider.py` | OpenAI-compatible provider (handles 12+ backends) |
| `ollama_provider.py` | Local Ollama wrapper |
| `vision_provider.py` | Dedicated vision LLM with API key cycling (rate limit distribution) |
| `prompt.py` | System prompt generation from presets + relationship + user profile + desktop commands |
| `response_parser.py` | Tag extraction (`[ACTION]`, `[DESKTOP]`, `[DIRECTIVE]`, etc.) + TTS text sanitization |

### desktop_pet/ — GUI
| File | Owns |
|------|------|
| `pet_window.py` | Main transparent frameless always-on-top window, sprite rendering, roaming, drag |
| `pet_controller.py` | Thread-safe Qt signal bridge: pipeline thread → main thread |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maresmaremares/bonziPONY](https://github.com/maresmaremares/bonziPONY) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
