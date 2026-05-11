---
trigger: always_on
description: **Purpose**: Primary maintenance guide for AI assistants maintaining the Flipper Wedge Flipper Zero application.
---

# CLAUDE.md - Flipper Wedge Maintenance Guide

**Purpose**: Primary maintenance guide for AI assistants maintaining the Flipper Wedge Flipper Zero application.

**Focus**: Stability, cross-firmware compatibility, regression prevention.

---

## Quick Navigation

- **[ARCHITECTURE_PATTERNS.md](docs/ARCHITECTURE_PATTERNS.md)** - Critical code patterns and architecture details
- **[QUICK_REFERENCE.md](docs/QUICK_REFERENCE.md)** - Commands, troubleshooting, quick fixes
- **[BUILD_MULTI_FIRMWARE.md](BUILD_MULTI_FIRMWARE.md)** - Build and deploy workflows
- **[FIRMWARE_COMPATIBILITY.md](docs/FIRMWARE_COMPATIBILITY.md)** - Tested versions and compatibility
- **[README.md](README.md)** - User documentation

---

## Project Status

**Current Version**: 1.0 (Feature-Complete)
**Status**: Active Maintenance
**Target**: Stability and multi-firmware compatibility

### Core Features (Complete)
✅ RFID (125 kHz) reading - EM4100, HID Prox, Indala
✅ NFC (13.56 MHz) reading - ISO14443A/B, MIFARE, NTAG
✅ NDEF parsing - Type 2, Type 4, Type 5 text records
✅ USB HID keyboard output
✅ Bluetooth HID keyboard output
✅ 5 scanning modes (NFC, RFID, NDEF, NFC+RFID, RFID+NFC)
✅ Dynamic USB/BLE switching (no app restart required)
✅ Configurable settings (delimiter, Enter key, output mode, vibration)
✅ Settings persistence
✅ Scan logging to SD card
✅ Haptic/LED/sound feedback
✅ Mode startup behavior (remember last mode or default)

### Supported Firmwares
- **Official** (Primary) - flipperzero-firmware
- **Unleashed** - unleashed-firmware
- **Momentum** (includes Xtreme) - Momentum-Firmware
- **RogueMaster** (Secondary) - roguemaster-firmware

---

## Maintenance Philosophy

### Primary Goals
1. **Stability First**: No new features without explicit user request
2. **Cross-Firmware Compatibility**: Test on all 4 firmwares before release
3. **Regression Prevention**: Validate all existing features after any change
4. **API Compatibility**: Monitor firmware API changes and adapt proactively
5. **User Experience**: Maintain consistent behavior across firmware versions

### What NOT to Do
- ❌ Add new features unless explicitly requested
- ❌ Refactor working code without a clear maintenance benefit
- ❌ Optimize code that isn't causing problems
- ❌ Change UX patterns without user feedback
- ❌ Update firmware dependencies without thorough testing

### When to Make Changes
- ✅ Firmware API breaks compatibility (required adaptation)
- ✅ Critical bugs affecting core functionality
- ✅ Security vulnerabilities
- ✅ User-reported issues with evidence
- ✅ Explicitly requested enhancements from maintainers

---

## Build & Deploy Workflows

See **[BUILD_MULTI_FIRMWARE.md](BUILD_MULTI_FIRMWARE.md)** for detailed build instructions.

### Quick Reference

```bash
# Build for specific firmware
./build.sh [official|unleashed|momentum|roguemaster]

# Build and deploy to connected Flipper
./deploy.sh [firmware]

# Build for all firmwares
./build-all-firmwares.sh
```

**Output**: `dist/<firmware>/<version>/flipper_wedge.fap`

### Firmware Update Detection

Build scripts automatically detect firmware version changes:

```
⚠️ FIRMWARE VERSION CHANGED: release → tag 1.3.5 (a1b2c3d)
```

**When you see this**:
1. Read firmware changelog for API changes
2. Run regression tests after building
3. Check for deprecation warnings
4. Test on actual hardware before distribution

---

## Testing & Validation Protocols

### Pre-Release Testing Checklist

Before releasing any update, **ALL** tests must pass on **ALL** supported firmwares.

#### Build Validation
- [ ] Official firmware: builds without warnings
- [ ] Unleashed firmware: builds without warnings
- [ ] Momentum firmware: builds without warnings
- [ ] RogueMaster firmware: builds without warnings (or document known issues)

#### Core Functionality Tests

**NFC Reading** (test on each firmware)
- [ ] ISO14443A tag UID read correctly
- [ ] 4-byte UID formatted correctly
- [ ] 7-byte UID formatted correctly
- [ ] Tag removal detected
- [ ] Multiple consecutive scans work

**RFID Reading** (test on each firmware)
- [ ] EM4100 tag UID read correctly
- [ ] HID Prox tag works (if available)
- [ ] Tag removal detected
- [ ] Multiple consecutive scans work

**NDEF Parsing** (test on each firmware)
- [ ] Type 2 NDEF text record parsed (NTAG)
- [ ] Type 4 NDEF text record parsed (if available)
- [ ] Type 5 NDEF text record parsed (if available)
- [ ] Non-NDEF tag shows error in NDEF mode
- [ ] Non-NDEF tag outputs UID in NFC/combo modes

**HID Output** (test on each firmware)
- [ ] USB HID connection detected
- [ ] USB HID types characters correctly
- [ ] USB HID Enter key works when enabled
- [ ] BT HID pairing works
- [ ] BT HID types characters correctly
- [ ] Dual output (USB + BT) works simultaneously
- [ ] "Connect USB or BT HID" shown when disconnected

**Scan Modes** (test on each firmware)
- [ ] NFC Only: scan → output UID → cooldown
- [ ] RFID Only: scan → output UID → cooldown
- [ ] NDEF Mode: scan NDEF tag → output text → cooldown
- [ ] NDEF Mode: scan non-NDEF → red LED → no output
- [ ] NFC+RFID: both scanned → combined output
- [ ] NFC+RFID: timeout after 5s if second tag missing
- [ ] RFID+NFC: both scanned → combined output

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DangerousThings/flipper-wedge](https://github.com/DangerousThings/flipper-wedge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
