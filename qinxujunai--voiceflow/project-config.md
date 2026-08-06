---
trigger: always_on
description: VoiceFlow is a Windows local-first dictation layer. Press **F2**, **Right Ctrl**, or a mouse side button to start recording, press again to stop, and cleaned text is copied to the clipboard and pasted at the current cursor. Press **Esc** to cancel.
---

# AGENTS.md -- VoiceFlow

## Project Identity

VoiceFlow is a Windows local-first dictation layer. Press **F2**, **Right Ctrl**, or a mouse side button to start recording, press again to stop, and cleaned text is copied to the clipboard and pasted at the current cursor. Press **Esc** to cancel.

The current target is stability: no orphan processes, working tray exit, complete final transcription, clipboard fallback, and truthful docs.

## Run

```bat
start.bat
venv\Scripts\python.exe src\main.py
```

## Verify

Do not use `src\*.py` with `py_compile` on Windows; Python does not expand that glob.

```bat
venv\Scripts\python.exe scripts\verify.py
```

Individual checks:

```bat
venv\Scripts\python.exe scripts\doctor.py
venv\Scripts\python.exe -m py_compile src\main.py src\overlay_webview.py src\hotkey_manager.py src\output_handler.py src\text_cleaner.py src\transcriber.py src\streaming_transcriber.py src\audio_capture.py src\recording_session.py src\vocabulary.py
venv\Scripts\python.exe -m pytest tests -q
venv\Scripts\python.exe test_integration.py
```

## Architecture

```text
src/
  main.py              # orchestration, lifecycle, streaming preview
  hotkey_manager.py    # keyboard + pynput mouse side buttons
  recording_session.py # recording lifecycle
  audio_capture.py     # sounddevice microphone adapter
  transcriber.py       # sherpa-onnx ASR
  streaming_transcriber.py # small online ASR for capsule preview
  vocabulary.py        # layered dictionary/corrections
  text_cleaner.py      # deterministic cleanup
  output_handler.py    # clipboard first, then Ctrl+V
  history_store.py     # logs/history.jsonl
  overlay_webview.py   # PyQt overlay + tray menu
  overlay.html         # centered pill UI
  tray_icon.py         # runtime status tray icon
```

## Product Rules

- Offline by default. Do not add cloud ASR, cloud LLM, or hidden network calls.
- Never lose text. If text exists, it must remain in clipboard and `logs/history.jsonl`.
- Do not restore the previous clipboard after dictation.
- Final output must cover the complete stopped audio; streaming preview is only preview.
- Streaming preview feeds each new PCM sample once to its own online recognizer.
- Never restore rolling-window preview through the final SenseVoice recognizer:
  overlapping retranscription causes delayed chunks and hypothesis rollback.
- Long recordings may progressively cache stable final segments during recording, but stop-time output must still include the remaining tail and cover the complete audio.
- Default triggers are `f2`, `right_ctrl`, `xbutton1`, and `xbutton2`. Do not add combo keys as defaults — suppress=True blocks the individual keys from normal use.
- Tray right-click menu must keep a working `退出` action.
- Keep the overlay small, centered, and quiet.
- If docs disagree with runtime behavior, fix the docs or the code immediately.

## Hotkeys

```yaml
hotkeys:
  push_to_talk: ["f2", "xbutton1", "xbutton2", "right_ctrl"]
  cancel: "escape"
```

All push-to-talk keys are single keys — no combo keys. `f2` uses the `keyboard` package and is suppressed. `right_ctrl` uses `pynput` so left/right Ctrl stay distinct. Mouse side buttons use `pynput` and are not suppressed.

### Key Reference

| Key | Type | Notes |
|---|---|---|
| `f2` | keyboard | Windows default |
| `right_ctrl` | keyboard | detected via pynput virtual key code |
| `xbutton1` | mouse | side button (back) |
| `xbutton2` | mouse | side button (forward) |
| `escape` | keyboard | cancel recording |

### Right Ctrl Implementation

`right_ctrl` is detected via `pynput.keyboard.Listener`, not the `keyboard` library. pynput uses virtual key codes (`VK_RCONTROL` = 0xA3) which are distinct from `VK_LCONTROL` even on keyboards where left/right Ctrl share the same scan code. The `keyboard` library cannot do this because it relies on scan codes only. Since right ctrl is rarely used in typing combos, no suppression is needed — the key press itself produces no character output, and the 0.5s debounce prevents accidental double-fires when used in combinations.

### Anti-pattern: Combo Keys

Never add combo keys like `ctrl+shift+space` via `keyboard.add_hotkey` with `suppress=True`. The `keyboard` library will suppress *every individual key in the combo* (Ctrl, Shift, Space), breaking copy/paste, input methods, and normal typing.

## Output Contract

The output path is:

```text
clean text -> pyperclip.copy(text) -> Ctrl+V -> history.jsonl
```

This is intentional. Even if `Ctrl+V` lands nowhere, the user can manually paste.

## Vocabulary

Primary files:

- `knowledge-base/builtin-ai.txt`
- `knowledge-base/corrections.txt`
- `knowledge-base/user-dictionary.txt`
- `knowledge-base/phrases.txt`

Legacy migration contract:

- v1 `ai-terms.txt`, `company-terms.txt`, and `user-custom.txt` are migration
  inputs only. They must not ship or appear in `hotwords.files`.
- Runtime schema v2 removes retired seed hashes, imports non-seed additions
  from modified legacy files, and then deletes the legacy filenames.

Use `wrong=correct` only in correction files.

## Packaging


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qinxujunai/VoiceFlow](https://github.com/qinxujunai/VoiceFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
