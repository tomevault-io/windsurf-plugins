---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Status

Functional v0. RDP clients (mstsc, Microsoft Remote Desktop, FreeRDP) can:
- Connect over TLS to the Mac on port 3390 with a local Mac username/password.
- See the primary display at native resolution with incremental damage-region updates.
- Optionally stream the display as **H.264 over EGFX** (`--enable-h264`, AVC420, Annex-B framing, VideoToolbox-encoded) — far less bandwidth than legacy bitmaps. Verified rendering on mstsc, on FreeRDP built with H.264 decode, and on the macOS Windows App / Microsoft Remote Desktop client (it decodes AVC420 over EGFX — only its *legacy* bitmap-codec list is NSCodec-only). Clients that genuinely don't advertise AVC420 decode (e.g. a decoder-less FreeRDP build) fall back to legacy BitmapUpdate automatically. **Caveat:** reconnecting *mstsc* to a still-running macrdp can show a blank screen (mstsc-specific EGFX surface-handling quirk — confirmed not a server bug, since FreeRDP reconnects cleanly); workaround is to restart macrdp or fully reopen the mstsc window. See the H.264 quirk note below.
- Drive keyboard and mouse, including modifier keys (per-side L/R tracking with NX_DEVICE bits, Caps Lock as a toggle, MS-RDPBCGR Synchronize lock-state reconciliation), mouse buttons, and wheel.
- Forward macOS symbolic hotkeys that WindowServer's dispatcher refuses to fire for user-space CGEventPost: Cmd+Tab / Cmd+Shift+Tab cycle apps via Accessibility API (per-bundle dedup with MRU, dead-pid filtering via `kill(pid, 0)`), Cmd+\` / Cmd+Shift+\` cycle windows of the current app (AXRaise + window AXMain + app AXMainWindow for Electron compatibility), Cmd+Space invokes Spotlight via AppleScript, Cmd+Shift+3/4/5 shell out to `/usr/sbin/screencapture` or open Screenshot.app.
- See the real macOS cursor shape (I-beam, hand, etc.) overlaid by the client.
- Copy/paste UTF-8 text and images (CF_DIB ↔ PNG) between Mac and remote.
- Mac→Windows file copy, including whole folders: copying a file or directory in Finder and pasting on Windows produces a real file/tree in Explorer. The pasteboard walk recurses into directories (skipping symlinks, capped at 10 000 descriptors per copy) and emits one FILEGROUPDESCRIPTORW entry per leaf with `relative_path` set so upstream's wire encoder reconstructs the right `MyFolder\sub\file.txt` cFileName. Bytes stream via MS-RDPECLIP `FileContentsRequest` SIZE + RANGE chunks (4 MiB per chunk). Reaches upstream `Cliprdr::initiate_file_copy` via the vendored `ServerEvent::ClipboardFileCopy(Vec<FileDescriptor>)` variant — that's the only API that populates `local_file_list`, without which upstream short-circuits every byte fetch with CB_RESPONSE_FAIL. Finder hands out *file-reference* URLs (`/.file/id=...`); we resolve them through `NSURL::URLByResolvingSymlinksInPath` because `std::fs::metadata` can't stat them directly.
- Windows→Mac file copy (one or more files; recursive folder copy via Ctrl-C does *not* work — see caveat below): when Windows announces a `FileGroupDescriptorW` we **eagerly** download every entry to `/tmp/macrdp-paste-<pid>-<nanos>/` via parallel `FileContentsRequest` chunks (1 MiB × 8 in flight), recreating any directory structure encoded in each descriptor's `relative_path`, then publish the top-level entries to NSPasteboard as real `NSURL`s. The eager approach is forced because Cocoa's `NSFilePromiseProvider` / `NSFilePromiseReceiver` is drag-and-drop-only — Finder's Cmd-V never calls into a promise delegate. `resolve_dest` path-sanitizes every `relative_path` component (rejects `.`, `..`, embedded `/`) so a malicious remote can't escape the temp sandbox. When the download lands we play `/System/Library/Sounds/Glass.aiff` (`afplay` bypasses notification permissions; `osascript display notification` was silently suppressed because macOS attributes the banner to the unsigned macrdp binary) and, *only if Finder is the frontmost app*, fire `Cmd-V` via System Events so the paste the user attempted finishes automatically. A `SelfChangeCount` atomic stops our own pasteboard write from being rebroadcast to Windows by the change-count poller.

  **Ctrl-C on a folder in Windows Explorer is a known no-op** — not our bug, and not fixable from the server side. Explorer puts `CFSTR_SHELLIDLIST` (Shell IDList Array) on the clipboard as the primary format and delay-renders `FileGroupDescriptorW` only when a shell-aware receiver asks. mstsc doesn't request the delayed format, so it never forwards anything via CLIPRDR — `cliprdr=debug` shows zero PDUs for the folder copy attempt. Workaround for the user: enter the folder in Explorer, `Ctrl-A` then `Ctrl-C` to copy the contents (with directory descriptors for any subfolders) — that path uses `FileGroupDescriptorW` directly and forwards correctly. True drag-from-Windows folder copy would need drive redirection (a different RDP feature, not clipboard).
- Forward macOS system audio to the remote (RDPSND, 44.1 kHz stereo 16-bit PCM; SCK captures at 48 kHz and the capture loop resamples via `rubato`).
- NLA / CredSSP authentication — no more "type username before Connect" mstsc workaround.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clintcan/macrdp](https://github.com/clintcan/macrdp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
