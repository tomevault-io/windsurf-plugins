---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ⛔ CRITICAL: USB Device Safety — READ FIRST

**HiDock USB devices lock up when improperly accessed. The device is NOT corrupted — the USB interface gets stuck in "active" state, rejecting new connections until drained or physically restarted. This has happened multiple times and is UNACCEPTABLE.**

### NEVER DO:
- **Multiple `open()`/`close()` cycles** in rapid succession
- **Exploratory USB scripts** ("let me try `findByIds()` and see what happens")
- **Diagnostic probing** (testing different USB APIs, checking endpoints, dumping descriptors)
- **Retrying failed connections** — if `LIBUSB_ERROR_ACCESS` appears, STOP IMMEDIATELY
- **Switching between USB APIs** (WebUSB vs native `usb` vs PyUSB) on the same device
- **Any USB code that isn't the final, tested implementation**
- **Using `endpoint.transfer()` in a manual read loop** — causes data loss on Windows due to `BlockingCall` gap in the USB event thread. Use `endpoint.startPoll()` instead (see below).

### ALWAYS DO:
- **Test ALL USB code with mocks first** — unit tests, never real hardware
- **ONE clean connection attempt** when code is ready
- **ONE proper cleanup** (`stopPoll()` → wait for `'end'` → `release(true)` → `close()`)
- **If `LIBUSB_ERROR_ACCESS`: try drain first** (see recovery below), then ask user to power-cycle only if drain fails
- **Use `startPoll(3, 32768)` for reading** — keeps 3 transfers pending in the kernel. This is the ONLY correct way to do continuous USB reads with the npm `usb` package on Windows. The `3` means 3 simultaneous transfers; `32768` = `wMaxPacketSize * 64`.
- **Use `stopPoll()` for disconnect** — cancels all pending transfers cleanly. Listen for the `'end'` event before calling `release()`/`close()`.

### Recovery — USB Drain:
If the device enters `LIBUSB_ERROR_ACCESS` state, attempt this drain before asking for physical restart:
```javascript
dev.open();
iface.claim();
epIn.timeout = 1000;
// Read until timeout error (queue empty)
epIn.transfer(51200, callback); // repeat until err
iface.release(true, () => dev.close());
```
This clears any pending USB transfers and often recovers the device without power cycle.

### Jensen Protocol — File List Behavior:
1. Send `CMD_GET_FILE_LIST` (cmd=4)
2. Device takes **~90 seconds** to prepare and send all data (for 1400 files)
3. First response: Jensen message (cmd=4) with body starting `0xFF 0xFF` + 4-byte total count + file entries
4. Subsequent responses: Jensen messages (cmd=4) with more file entries
5. All responses have valid Jensen headers (`0x12 0x34`)
6. Final message has `bodyLength=0` = end of list
7. **Multiple transfers MUST be pending** — use `startPoll(3, 32768)`, not `transfer()`
8. **Header length field is 24-bit** — upper byte of the 4-byte length field is checksum length, lower 3 bytes are body length. `bodyLen = rawLen & 0x00FFFFFF`

### Why:
The HiDock USB controller enters a locked state (interface still marked "active") when subjected to rapid open/close cycles or concurrent access attempts from different USB stacks. Once in this state, ALL programs (browser, Python, Node.js, even the official HiNotes site) fail with "Access denied" until drain or physical power cycle. The user has used HiDock for over a year without this issue — every occurrence was caused by AI agents doing USB probing.

---

## Project Overview

**HiDock Next** is a **universal knowledge hub** - an integrated suite of applications that extracts insights, manages information, and produces results from ANY knowledge source (recordings, PDFs, PPTX, DOCX, MD, notes, calendar, email, Slack, and more).

### The Suite Evolution

The project evolved through four iterations, each building toward the ultimate vision:

1. **Desktop App** (`apps/desktop/`) - **First iteration: Device management focused**
   - Python/CustomTkinter GUI for managing HiDock® devices (H1, H1E, P1 models)
   - USB communication via Jensen protocol (PyUSB)
   - File sync, device settings, storage management
   - **Entry point**: Where users typically discover HiDock Next
   - **Focus**: Hardware management and basic file operations

2. **Web App** (`apps/web/`) - **Second iteration: Transcription focused**
   - React/TypeScript browser interface using WebUSB
   - AI transcription with multiple providers (Gemini, OpenAI, etc.)
   - Web-based device access (no drivers needed)
   - **Focus**: Making recordings accessible and transcribable anywhere

3. **Audio Insights** (`apps/audio-insights/`) - **Third iteration: Insights prototype**
   - AI-powered audio analysis tool
   - Extracting insights from transcriptions
   - **Focus**: Proving the concept of knowledge extraction from audio

4. **Electron App** (`apps/electron/`) - **Fourth iteration: The integrated hub** ⭐ **CURRENT FOCUS**
   - **Vision**: Universal knowledge hub integrating all previous capabilities
   - **Scope**: Not just audio, but ANY artifact as a knowledge source
   - Knowledge sources: recordings, PDFs, presentations, documents, markdown, notes, calendar events, emails, Slack messages, etc.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sgeraldes/hidock-next](https://github.com/sgeraldes/hidock-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
