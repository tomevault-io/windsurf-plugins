---
trigger: always_on
description: Scaffold a two-language tutor in any project — Claude speaks the target language aloud (with IPA) while notes and corrections in your native language stay silent. Use when the user asks to learn or practice a foreign language with spoken feedback, or when they want Claude's target-language responses read aloud in Claude Code.
---


# claude-speech

This skill bootstraps a self-contained language-learning project inside the user's current Claude Code workspace. It works with **two languages**: a **target language** (the one being learned — spoken aloud, with IPA pronunciation help) and a **common language** (the learner's native tongue — used for notes, corrections, and free chat, never spoken). It installs:
- a teacher persona (`CLAUDE.md`) that speaks the target language and writes all notes in the common language,
- a `Stop` hook + `scripts/speak_lang.py` that uses `edge-tts` to speak only the target-language portion of Claude's replies aloud,
- a `UserPromptSubmit` hook + `scripts/push_to_talk.py` + `scripts/inject_transcript.py` for **two-key push-to-talk voice input** — hold **F9** to speak the target language or **F10** to speak the common language. The held key forces the transcription language (no auto-detection, so mixed-language speech isn't misread), transcribes via local Whisper, adds an IPA line (espeak-ng) only for target-language speech, and pastes the result into the chat as your message.

## When to use

Trigger when the user says any of:
- "let's practice {language}"
- "teach me {language}"
- "set up a {language} tutor"
- "I want Claude to speak {language} responses"
- "scaffold claude-speech for {language}"

## How to invoke

0. **Handle control arguments first.** Before any of the steps below, inspect the argument the user passed:
   - If the argument is `off`, `stop`, or `kill` (case-insensitive): skip all install and scaffold steps. Take these actions in order:
     1. Find and terminate every running `push_to_talk.py` daemon **and any `selection_toolbar.py` process**. On Windows the reliable command is (note the **single-quoted** `-Command` argument so this also works when invoked from bash/zsh — double quotes would let the shell interpolate `$_` and `$(...)` before PowerShell sees them):
        ```
        powershell -NoProfile -Command 'Get-CimInstance Win32_Process | Where-Object { $_.Name -in @("py.exe","python.exe","pythonw.exe") -and ($_.CommandLine -like "*push_to_talk.py*" -or $_.CommandLine -like "*selection_toolbar.py*") } | ForEach-Object { Write-Host "killed PID $($_.ProcessId)"; Stop-Process -Id $_.ProcessId -Force }'
        ```
        **Crucially: the filter MUST require the process Name to be one of `py.exe` / `python.exe` / `pythonw.exe`.** Without that restriction, the filter also matches shell wrappers that happen to have `push_to_talk.py` literally in their command line (e.g. the very PowerShell invocation you're running) and will pollute the result list.
     2. Terminate the resident `whisper-server.exe` the daemon started (it loads the model into VRAM and does **not** die with the daemon when force-killed). Match only **this project's** server by requiring its command line to reference the project's `tools\whisper.cpp` path, so other projects' servers are left running:
        ```
        powershell -NoProfile -Command 'Get-CimInstance Win32_Process | Where-Object { $_.Name -eq "whisper-server.exe" -and $_.CommandLine -like "*<project_root>\tools\whisper.cpp*" } | ForEach-Object { Write-Host "killed server PID $($_.ProcessId)"; Stop-Process -Id $_.ProcessId -Force }'
        ```
        Substitute `<project_root>` with the actual project directory. (If voice-in was never set up, or the daemon never started, the filter simply matches nothing.)
     3. Delete `<project_root>/recordings/latest_transcript.txt` if it exists, so the UserPromptSubmit hook doesn't keep re-injecting the last transcript on subsequent manual Enters now that the daemon isn't writing fresh ones.
     4. **Disable spoken output.** Run `py toggle_voice.py --project-dir <project_root> --off` (from this skill's directory). This is what actually silences replies: it surgically removes the `speak_lang.py` Stop hook from `.claude/settings.json` and stashes an exact copy in `.claude/speak_lang.hook.json` so it can be restored later without re-running the installer. The mic daemon (steps 1–2) and spoken output are independent switches — killing the daemon alone leaves the Stop hook firing, so this step is required for a full "off". It's idempotent: a no-op if voice was already off.
     5. Report the PIDs killed (or "no daemons were running"), confirm the stale transcript was cleared, and confirm spoken output is now off.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dimoniada/claude-speech-skill](https://github.com/Dimoniada/claude-speech-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
