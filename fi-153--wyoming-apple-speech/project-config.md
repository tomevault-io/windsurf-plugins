---
trigger: always_on
description: A Wyoming protocol STT + TTS server that bridges macOS on-device speech recognition (Apple's
---

# Wyoming Apple Speech

A Wyoming protocol STT + TTS server that bridges macOS on-device speech recognition (Apple's
Speech framework) and Siri text-to-speech to Home Assistant's Voice pipeline.

## Project Components

- **swift/Sources/AppleSTT/** — Swift CLI tool (`apple-stt`) that reads PCM audio from stdin and
  outputs transcribed text as JSON.
  Uses SpeechAnalyzer on macOS 26+ and SFSpeechRecognizer on older systems.
  With `--worker` it runs as a persistent streaming worker instead: framed JSON commands on
  stdin (`transcribe` → `audio`+PCM payload → `stop`), `ready`/`partial`/`final`/`error`
  JSON lines on stdout (`WorkerMode.swift`, `StreamingSession.swift`). The binary must be
  re-signed after building (`codesign -f -s -`, done by the Makefile) so the embedded
  Info.plist's speech-recognition usage description is bound into the signature — TCC
  SIGABRTs the process otherwise. When launched from a terminal/IDE shell, TCC attributes
  the speech-recognition request to the *responsible process* (the terminal), so test
  worker mode via launchd (`launchctl submit`) or the installed service.
  - `LocaleMatching.swift` — pure function `bestMatchingLocale(for:in:)` for resolving bare
    language codes (e.g. "en") to full locales (e.g. "en-US") against `SpeechTranscriber.supportedLocales`.
  - `SupportedLanguages.swift` — pure function `languageCodes(from:)` for extracting deduplicated,
    sorted short language codes from a list of locales. Used by `--list-languages` CLI flag.
  - `swift/Tests/AppleSTTTests/` — Swift unit tests (Swift Testing framework).
- **swift/Sources/AppleTTS/** — Swift CLI tool (`apple-tts`) exposing the private Siri synthesis
  engine (`SiriTTSService.framework`). Runs as a long-lived worker: JSON commands on stdin,
  JSON-header + binary PCM frames on stdout. `--list-voices` prints the system-managed Siri
  voices (the only ones that reliably load; see `context/planning/add-siri-tts-streaming.md`
  for the engine's lifecycle constraints — engines are never deallocated, init failures are
  process-fatal by design).
  - `swift/Tests/AppleTTSTests/` — voice-specifier parsing and asset-discovery tests.
- **wyoming_apple_speech/** — Python Wyoming protocol server. Handles TCP connections from Home
  Assistant. For STT, `stt.py` keeps a pool of pre-warmed `apple-stt --worker` processes
  (mirroring the TTS pool design); the handler streams audio chunks into a worker session as
  they arrive, forwards partial transcripts as `transcript-chunk` events, and emits the final
  transcript right after `audio-stop`. Any streaming failure falls back to the buffered
  one-shot transcription path (audio is always buffered in parallel). For TTS, `tts.py`
  keeps a pool of pre-warmed `apple-tts` workers (acquire triggers a background replacement
  spawn) and the handler serves both legacy `synthesize` and streaming
  `synthesize-start/chunk/stop` with sentence-level incremental synthesis.
- **scripts/** — Install and uninstall scripts for the launchd service.
- **packaging/** — Release tooling: `build-release-tarball.sh` produces the
  GitHub release artifact; `formula.rb.template` + `python-resources.rb` are
  rendered by CI into the Homebrew formula.

## Key References

- **Code style:** [`context/styling/formatting.md`](context/styling/formatting.md) — naming, formatting, docstrings for Swift and Python

## Make Targets

All development commands go through the Makefile. **Always use `make` commands** instead of
invoking tools directly.

```bash
make            # Show help with all available targets
make venv       # Create Python venv and install dev dependencies
make build      # Build the Swift CLI binary
make test       # Run Python tests (creates venv if needed)
make swift-test # Run Swift unit tests (LocaleMatching, etc.)
make run        # Run the server locally (builds everything first)
make stop       # Stop the server running on PORT
make install    # Install as launchd service
make uninstall  # Remove the launchd service and files
make clean      # Remove build artifacts and venv
```

**Important**: Never run `pytest`, `swift build`, or `pip install` directly. Always use the
corresponding `make` target. This ensures the correct venv, paths, and build flags are used.

## Planning Workflow

All plans live under `context/planning/`. The design plan and implementation plan for a given task
**must** be in the same file — design first, implementation checklist appended below after approval.

Whenever the user asks Claude to plan a task, Claude **must** write the plan as a `.md` file inside
`context/planning/` before doing any implementation work. File names must be descriptive kebab-case
(e.g., `implement-upload-endpoint.md`). After writing the plan, Claude notifies the user of the file
path and waits for explicit approval before proceeding. The user may edit the plan file directly
using `/user <comment>` annotations; When new additions to the plan are made in response to comments
mark them with `/new`; Delete all the `/new` already present in a plan when updating or adding the
todo list; implementation begins only after the user explicitly approves. Plans are kept locally
and excluded from git via .gitignore.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FI-153/wyoming-apple-speech](https://github.com/FI-153/wyoming-apple-speech) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
