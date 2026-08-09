---
trigger: always_on
description: A Windows system-tray app that detects Microsoft Teams calls, records system + microphone
---

# Copilot instructions — MeetingScribe

A Windows system-tray app that detects Microsoft Teams calls, records system + microphone
audio, transcribes locally with Whisper, and writes the transcript as a Markdown note into
an Obsidian vault. Everything runs offline and free — no cloud services, no API keys.

## Repo shape

```
build.ps1                   Single-file publish + smoke test (the build entry point that matters)
MeetingScribe.slnx
.github/
  workflows/release.yml     Tag v* -> build on a Windows runner -> GitHub Release
  release-notes-template.md Release body, with {{VERSION}}/{{SIZE}}/{{SHA}} placeholders
src/MeetingScribe/
  Program.cs                Entry point: single-instance mutex, exception hooks, CLI dispatch
  TrayApplicationContext.cs NotifyIcon, menu, wiring
  Configuration/            AppConfig (settings model), ConfigStore (JSON load/save + vault guess)
  Detection/                Two independent call-detection probes + CallMonitor debouncer
  Recording/                Dual-track WASAPI capture
  Transcription/            Audio conversion to 16 kHz mono + Whisper.net wrapper
  Notes/                    Markdown + YAML frontmatter rendering
  Pipeline/                 MeetingPipeline orchestration, AppState
  Diagnostics/              SelfTest — the CLI verification harness
  Infrastructure/           Log, Paths, FileNames, TrayIcons, StartupRegistration
```

Runtime state lives in `%APPDATA%\MeetingScribe\` (`config.json`, `logs\`, `models\`,
`recordings\`). Nothing user-specific is stored in the repo.

## Build, run, verify

```powershell
dotnet build                                  # fast inner loop
dotnet run --project src\MeetingScribe        # launches the tray app
.\build.ps1                                   # single self-contained publish\MeetingScribe.exe
```

`build.ps1` is always self-contained on purpose. A framework-dependent single-file build is
*larger* (~122 MB vs ~84 MB) because `EnableCompressionInSingleFile` only applies to
self-contained publishes. Don't "optimise" that back.

There is **no test project**. Verification is done through the CLI harness in
`Diagnostics/SelfTest.cs`:

```powershell
MeetingScribe.exe --version              # cheap, no side effects (build.ps1 smoke test uses this)
MeetingScribe.exe --help
MeetingScribe.exe --selftest 10          # devices, both detection probes, a 10s two-track recording
MeetingScribe.exe --transcribe <file>    # conversion + Whisper + note preview into a sandbox vault
```

Any argument starting with `--` routes to `SelfTest.RunAsync` and attaches to the parent
console. Reports are written to `%TEMP%\meetingscribe-selftest.txt`.

**Always verify audio/transcription changes by actually running the harness.** Synthesising
test speech is easy and beats guessing:

```powershell
Add-Type -AssemblyName System.Speech
$s = New-Object System.Speech.Synthesis.SpeechSynthesizer
$s.SetOutputToWaveFile("$env:TEMP\t.wav"); $s.Speak("test sentence"); $s.Dispose()
```

## Hard-won gotchas — do not regress these

**Loopback capture goes silent.** `WasapiLoopbackCapture` delivers *no* buffers when nothing
is playing, which desyncs the two tracks. `MeetingRecorder.StartSilenceKeepAlive` fixes this
by playing a `SilenceProvider` through `WasapiOut`. It must use the *device mix format*, not
an arbitrary one, or `WasapiOut` rejects it.

**Two tracks, not one.** Mic = "Me", loopback = "Participants". This gives speaker
attribution with no diarization model. Don't collapse it into a single mixed track.

**Whisper hallucinates on silence** ("Thank you.", "[BLANK_AUDIO]"). Three independent
filters guard against this: an RMS loudness envelope computed during conversion
(`SilenceRmsThreshold`), `MinimumProbability`, and a literal/regex hallucination list. Keep
all three.

**Speaker bleed duplicates every line.** With speakers instead of a headset the mic re-records
the meeting, so both tracks transcribe the same sentence. `Transcription/EchoFilter` drops the
mic copy by word-overlap against time-overlapping participants segments. Signal-domain AEC was
rejected: the two WASAPI endpoints have independent clocks, so the echo delay drifts over a
long meeting and a fixed-delay canceller falls apart.

**Window captions are pipe-delimited, and the title is not always segment zero.** While a call
is being joined Teams uses "Meeting join | Real title | Microsoft Teams". `MeetingTitleResolver`
takes the first segment that isn't boilerplate, not the first segment.

**Detection must not depend on the Teams UI.** Both probes are deliberately UI-independent so
Teams updates cannot break them:

1. WASAPI — active audio sessions on capture endpoints, PID matched against a process regex.
2. Registry — `HKCU\...\CapabilityAccessManager\ConsentStore\microphone`; a
   `LastUsedTimeStop` of `0` means in use *right now*. Packaged apps are direct subkeys;
   unpackaged live under `NonPackaged` with `#` substituted for `\`.

Either probe alone is enough to trigger. Widening `Detection.ProcessNamePattern` is how you
add Zoom/Slack/browser calls — no code change needed.

**Never dispose `Process` objects inside a LINQ predicate** that a later `.Select` still
reads from. This shipped as a real bug in `MeetingTitleResolver`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [edandersen/teams-meeting-scribe](https://github.com/edandersen/teams-meeting-scribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
