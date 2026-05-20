---
trigger: always_on
description: Guidance for AI coding agents (Claude Code, Codex, etc.) working in this repository.
---

# AGENTS.md

Guidance for AI coding agents (Claude Code, Codex, etc.) working in this repository.

## Project at a glance

`SharePointVideoDownloader` is a small Windows-focused C# console app (.NET 9) that helps a logged-in user download Microsoft SharePoint / Stream videos (typically Teams meeting recordings).

The app drives a real Chromium instance with **PuppeteerSharp** and supports two strategies:

**Default — direct file download.** Used unless `-c / --capture` is passed.
1. Navigate to the SharePoint `stream.aspx?id=<path>` URL.
2. Parse the `id` query parameter to derive the underlying file path inside the user's OneDrive / SharePoint site.
3. Use `Page.setDownloadBehavior` over CDP to redirect Chromium downloads into the desired output directory.
4. Trigger the download by injecting an `<a download>` click pointing at `https://<host>/<path>?download=1`. Chromium handles auth, redirects, and `Content-Disposition` natively. The file that lands is the original non-DRM mp4.
5. Poll for `<file>.crdownload` (in-progress) and the final file to detect completion.
6. If the user passed `-a / --audio`, post-process via ffmpeg to extract an mp3 and delete the source mp4.

**`--capture` — in-browser screen + audio re-recording.** Used when direct download fails (403) or when the recording is view-only / DRM-protected.
1. Launch Chromium non-headless (DRM playback requires a real surface) but with `--window-position=-2400,-2400` so the window is invisible. `--use-fake-ui-for-media-stream` and `--auto-accept-this-tab-capture` suppress the getDisplayMedia picker. **Do NOT pass `--mute-audio`** — it mutes the captured audio as well, not just the speakers.
2. Navigate to the page and trigger play (mouse click on `<video>`, then Space, then synthetic JS click as fallbacks).
3. Inject a script that:
   - Hooks the `<video>` element's audio via `AudioContext.createMediaElementSource(video) → MediaStreamDestination` and never connects to `audioContext.destination`. Web Audio takes over routing entirely, so speakers stay silent. Audio capture is full-volume.
   - Probes `canvas.drawImage(video)` on a center 8×8 patch. If pixels come back non-zero, runs a `requestVideoFrameCallback` loop to draw frames at native `videoWidth × videoHeight` and exposes them via `canvas.captureStream()` — full source resolution, no UI chrome. If the probe is all-zero or throws (EME taint), falls back to `getDisplayMedia` tab capture cropped to the `<video>` element's bounds.
   - Combines the canvas video track + Web Audio audio track into a `MediaRecorder` (VP9 + Opus).
   - On `<video>.ended` (or after `--capture-seconds`) stops the recorder, builds a Blob, and triggers an `<a download>` click so the file lands in the directory configured via `Page.setDownloadBehavior`.
4. Wait for the file to land (`<file>.crdownload` → final).
5. ffmpeg post-processes:
   - `-c copy` remux to add a Cues element so the webm is seekable;
   - `libx264 + aac + faststart` transcode if the user requested `.mp4`;
   - `-vn libmp3lame` audio extraction if `-a / --audio`.
   ffmpeg is a soft dependency: the program prints a warning at startup if it cannot find ffmpeg, but capture still produces a raw webm.

Authentication is delegated to the browser session — the app never types passwords. The first run is non-headless so the user can sign in to Microsoft 365, and the session is persisted via PuppeteerSharp's `UserDataDir` (defaults to `%LOCALAPPDATA%\PuppeteerSession`).

## Repository layout

| Path | Purpose |
| --- | --- |
| `Program.cs` | The entire application — argument parsing, browser orchestration, direct download, capture, ffmpeg post-processing. Single-file by design. |
| `SharePointVideoDownloader.csproj` | .NET 9, references `PuppeteerSharp` only. |
| `SharePointVideoDownloader.sln` | Visual Studio solution. |
| `Dependencies/` | Folder copied verbatim into `*-DotNet-Dependencies` release ZIPs. Currently empty (used to hold a bundled `yt-dlp.exe`; we now rely on user-installed ffmpeg via PATH). |
| `Releases/` | Pre-built ZIPs published on GitHub Releases (DotNet, x64/x86/ARM64 self-contained). |
| `_publishAll.bat` | Windows build/zip. Hard-codes the version string. |
| `_publishAll.sh` | macOS / Linux build/zip. Same release flavours plus `osx-x64`, `osx-arm64`, `linux-x64`. |
| `README.md` | End-user documentation. |
| `LICENSE` | MIT. |

There is intentionally no `src/`, no test project, no service abstractions — adding any of those would be over-engineering for a tool this size.

## How the core flow is implemented (`Program.cs`)

Anchor points if you need to edit:

- **CLI parsing**: `Main` walks `args` manually with a `switch` on `-u/--url`, `-a/--audio`, `-o/--output`, `-c/--capture`, `--capture-seconds`, `-v/--visible`, `-h/--help`. Falls back to interactive prompts when arguments are absent or invalid. See `ShowHelp()` for the canonical surface.
- **ffmpeg startup check**: `WarnIfFfmpegMissing` (called from `Main` after argument parsing) prints a yellow heads-up if ffmpeg is not on PATH and not next to the exe. The program continues regardless; users get a webm without seek index, and `-a` becomes a no-op.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PatBQc/SharePointVideoDownloader](https://github.com/PatBQc/SharePointVideoDownloader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
