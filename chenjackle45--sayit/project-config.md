---
trigger: always_on
description: > Tauri v2 + Vue 3 + Rust 語音轉文字桌面應用
---

# SayIt — Claude Code 專案記憶檔

> Tauri v2 + Vue 3 + Rust 語音轉文字桌面應用
> 完整規則請讀：`_bmad-output/project-context.md`（261 條）

## Quick Reference

| 文件 | 路徑 | 用途 |
|------|------|------|
| 完整規則 | `_bmad-output/project-context.md` | 所有 AI Agent 實作規則（必讀） |
| UX/UI 規範 | `_bmad-output/planning-artifacts/ux-ui-design-spec.md` | UI 設計、色彩、元件規範 |
| 架構設計 | `_bmad-output/planning-artifacts/architecture.md` | 架構決策文件 |
| 設計稿 | `design.pen` | Pencil MCP 設計稿（UI 實作前必須先完成） |

## 雙視窗架構

```
 ┌─────────────────────────────────────────────────┐
 │                  Tauri Backend (Rust)            │
 │  lib.rs ─ plugins/ ─ clipboard_paste.rs         │
 │                      hotkey_listener.rs          │
 │                      keyboard_monitor.rs         │
 │                                                  │
 │  ┌─── invoke() ──┐     ┌─── emit() ────┐        │
 │  │               │     │               │        │
 │  ▼               ▼     ▼               ▼        │
 │ ┌──────────┐  ┌──────────────────────────┐      │
 │ │   HUD    │  │      Dashboard           │      │
 │ │ index.   │  │   main-window.html       │      │
 │ │ html     │  │   MainApp.vue + Router   │      │
 │ │ App.vue  │  │   4 views + DB + Store   │      │
 │ │ NotchHud │  │   shadcn-vue UI          │      │
 │ └──────────┘  └──────────────────────────┘      │
 │  label:main    label:main-window                │
 │  400x100       960x680 (min 720x480)            │
 │  transparent   decorations, resizable           │
 │  alwaysOnTop   預設隱藏                          │
 └─────────────────────────────────────────────────┘
```

## IPC 契約表

### Tauri Commands（Frontend → Rust）

| Command | Rust 位置 | 前端呼叫點 | 參數 | 回傳 |
|---------|-----------|-----------|------|------|
| `debug_log` | `lib.rs` | stores, main-window.ts | `level: String, message: String` | `()` |
| `request_app_restart` | `lib.rs` | main-window.ts | — | `()` |
| `update_hotkey_config` | `lib.rs` | useSettingsStore | `trigger_key: TriggerKey, trigger_mode: TriggerMode` | `Result<(), String>` |
| `get_hud_target_position` | `lib.rs` | — | `app: AppHandle` | `Result<HudTargetPosition, String>` |
| `paste_text` | `plugins/clipboard_paste.rs` | useVoiceFlowStore | `text: String` | `Result<(), ClipboardError>` |
| `copy_to_clipboard` | `plugins/clipboard_paste.rs` | HistoryView | `text: String` | `Result<(), ClipboardError>` |
| `capture_target_window` | `plugins/clipboard_paste.rs` | useVoiceFlowStore | — | `()` |
| `check_accessibility_permission_command` | `plugins/hotkey_listener.rs` | AccessibilityGuide.vue | — | `bool` |
| `open_accessibility_settings` | `plugins/hotkey_listener.rs` | AccessibilityGuide.vue | — | `Result<(), String>` |
| `reinitialize_hotkey_listener` | `plugins/hotkey_listener.rs` | AccessibilityGuide.vue | `app: AppHandle` | `Result<(), String>` |
| `reset_hotkey_state` | `plugins/hotkey_listener.rs` | useVoiceFlowStore | `state: State<HotkeyListenerState>` | `()` |
| `start_quality_monitor` | `plugins/keyboard_monitor.rs` | useVoiceFlowStore | `app: AppHandle` | `()` |
| `start_correction_monitor` | `plugins/keyboard_monitor.rs` | useVoiceFlowStore | `app: AppHandle` | `()` |
| `read_focused_text_field` | `plugins/text_field_reader.rs` | useVoiceFlowStore | — | `Result<Option<String>, String>` |
| `read_selected_text` | `plugins/text_field_reader.rs` | useVoiceFlowStore | — | `Result<Option<String>, String>` |
| `mute_system_audio` | `plugins/audio_control.rs` | useVoiceFlowStore | `state: State<AudioControlState>` | `Result<(), String>` |
| `restore_system_audio` | `plugins/audio_control.rs` | useVoiceFlowStore | `state: State<AudioControlState>` | `Result<(), String>` |
| `get_default_input_device_name` | `plugins/audio_recorder.rs` | SettingsView | — | `Option<String>` |
| `list_audio_input_devices` | `plugins/audio_recorder.rs` | SettingsView | — | `Vec<AudioInputDeviceInfo>` |
| `start_audio_preview` | `plugins/audio_recorder.rs` | SettingsView | `app: AppHandle, preview_state: State<AudioPreviewState>, device_name: String` | `Result<(), String>` |
| `stop_audio_preview` | `plugins/audio_recorder.rs` | SettingsView | `preview_state: State<AudioPreviewState>` | `()` |
| `start_recording` | `plugins/audio_recorder.rs` | useVoiceFlowStore | `app: AppHandle, state: State<AudioRecorderState>, device_name: String` | `Result<(), AudioRecorderError>` |
| `stop_recording` | `plugins/audio_recorder.rs` | useVoiceFlowStore | `state: State<AudioRecorderState>` | `Result<StopRecordingResult, AudioRecorderError>` |
| `save_recording_file` | `plugins/audio_recorder.rs` | useVoiceFlowStore | `id: String, app: AppHandle, state: State<AudioRecorderState>` | `Result<String, String>` |
| `read_recording_file` | `plugins/audio_recorder.rs` | HistoryView | `id: String, app: AppHandle` | `Result<Response, String>` |
| `delete_all_recordings` | `plugins/audio_recorder.rs` | SettingsView | `app: AppHandle` | `Result<u32, String>` |
| `cleanup_old_recordings` | `plugins/audio_recorder.rs` | main-window.ts | `days: u32, app: AppHandle` | `Result<Vec<String>, String>` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chenjackle45/SayIt](https://github.com/chenjackle45/SayIt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
