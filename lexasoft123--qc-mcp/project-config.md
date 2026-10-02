---
trigger: always_on
description: MCP server that controls a **Neural DSP Quad Cortex** over its reverse-engineered
---

# CLAUDE.md

MCP server that controls a **Neural DSP Quad Cortex** over its reverse-engineered
internal **USB-HID / protobuf** protocol (not MIDI). See `PROTOCOL.md` for the wire
protocol, `docs/DIRECTORY.md` for the preset/capture/IR catalog + scenes,
and `docs/COROS-4.1.md` for the 4.1 feature set (device presets, stomps,
Global EQ / I/O presets) with usage examples.
Supports **CorOS 4.0 and 4.1** — the schema is picked per connection from the
device's firmware (PROTOCOL.md §12) — on **macOS and Windows** (`docs/WINDOWS.md`).

## Layout
- `src/qc_mcp/`
  - `backend.py` — picks the HID transport for the OS (`open_hid`) and answers
    `direct_supported`/`bridge_supported`. All three backends below implement the
    same four methods: `open`/`set_report`/`read_reports`/`close`.
  - `iohid.py` — **macOS**: ctypes IOKit HID transport (input buffer must be
    report_size+1).
  - `winhid.py` — **Windows**: ctypes setupapi + hid.dll. Overlapped I/O; picks
    the collection with 129-byte reports; raises the driver's 32-report input
    queue to 512 (a directory dump overruns the default).
  - `bridge.py` — FIFO bridge: share Cortex Control's live session via the DYLD
    interposer (run MCP + the app at once). No handshake/heartbeat (the app owns them).
    **macOS only** — importing it elsewhere raises with that message.
  - `protocol.py` — framing (128-byte reports, flag bits, 8-byte command trailer,
    gzip), `COMMANDS`, encode/decode, `Reassembler`; **version negotiation**
    (`generation`/`set_version`/`supports`/`require`) + a descriptor pool per CorOS
    generation from `descriptors/qc_descriptors-<gen>.pb`.
  - `transport.py` — `QuadCortex`: open/handshake/heartbeat, read state, edit grid
    (add/delete block, params, per-scene params, splits/mixers, routing, bypass,
    captures, IRs), recall/save, list_directory.
  - `catalog.py` — `ModelRepo.xml` parser + value taper (log/linear, `to_norm`/
    `to_display`). `SYMBOLIC` resolves the ranges the XML leaves as names
    (`MIN_MIXER_DB` = -40, `MAX_MIXER_DB` = +12, calibrated against the app —
    only add a name once measured). Data attribution: neuraldsp.com/device-list.
  - `preset.py` — `describe(bp)` ⇄ `build(spec)` + `apply_spec` (spec ⇄ BinaryPreset).
  - `directory.py` — structure/search the on-device catalog (presets/IRs/captures).
    A file's slot is its **array position** (the `index` field is 0 in a whole
    read), so a setlist's 256 entries map straight to recall positions.
  - `leveling.py` — the preset-leveling bench Patchbay's Leveling view drives
    (`qc-mcp --leveling --socket …`, newline-JSON on stdio, attaches to the
    daemon like any other client). Reads/writes LaneOutputControl VOLUME in dB
    and streams `IOMeter`.
  - `server.py` — FastMCP server (~30 tools). `connect(mode=auto|bridge|direct)`:
    when nothing is running it RETURNS the mode options (relay the question to the
    user); `mode='bridge'` self-launches `interceptor/run-bridge.sh` (~20s cold) and
    joins; `mode='direct'` needs `quit_app=True` if Cortex Control holds the device.
    Other tools' `_conn()` still auto-detects a running bridge. Where bridge mode
    can't run, `auto` goes straight to direct instead of asking.
- `interceptor/` — DYLD interposer C + build/run scripts (capture + bridge). Logs and
  `catalog.json` are **gitignored** (contain library names / session ids).
- `tools/` — RE utilities (mostly macOS: they shell out to otool/codesign);
  `tools/win_hid_check.py` diagnoses a Windows setup (enumerate → open → round-trip,
  distinct exit codes per failure). `tools/gui/` — GUI-automation harness (below).
  After a CorOS update run all three: `interceptor/build.sh` (re-instrument the
  updated app), `tools/build_descriptors.py build <gen>` (new wire schema),
  `tools/dump_model_repo.py --diff` then without `--diff` (new device catalog).
- `.claude/skills/` — reusable reverse-engineering skills.

## Running
- `python3` alone lacks pyobjc; use `.venv/bin/python`. GUI tools auto-reexec into `.venv`.
- Tests (all offline, no device): `.venv/bin/python tests/test_directory.py`,
  `tests/test_protocol_versions.py`, `tests/test_tool_docs.py` (keeps the MCP
  self-describing — every gated feature must have a tool behind it),
  `tests/test_platform.py` (keeps the macOS and Windows backends interchangeable;
  it's the only check on `winhid.py` from a Mac).
- Device/GUI tools need the instrumented Cortex Control running (bridge) — see
  `interceptor/run-bridge.sh`.

## GUI harness + tests (`tools/gui/`)
Drives Cortex Control (screenshot + click) and correlates the interposer protocol log.
Needs **Claude.app** granted Screen Recording + Accessibility (macOS TCC).
**Capture and reads no longer touch the screen:** `shot` uses `screencapture -l
<winid>` (renders that window alone, occluded or parked off-screen), and `ax`
reads JUCE's accessibility tree — labelled controls, live values, exact frames,
no focus. Only clicking needs the screen (JUCE ignores AXPress and
CGEventPostToPid): `press "<name>"` borrows focus for ~1s and hands it back.
`park`/`home` move the window off every display and back.
- `gui.py` — `bounds`/`home`/`shot`/`click`/`type`/`key`/`act`/`decode`. **`home`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lexasoft123/qc-mcp](https://github.com/lexasoft123/qc-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
