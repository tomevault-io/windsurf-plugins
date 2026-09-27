---
trigger: always_on
description: This document provides comprehensive guidelines for AI assistants (and developers) working on the Growatt Modbus Home Assistant integration.
---

# Claude Development Guide - Growatt Modbus Integration

This document provides comprehensive guidelines for AI assistants (and developers) working on the Growatt Modbus Home Assistant integration.

---

## 🛑 RULES THAT PREVENT REAL BUGS

Every rule below exists because it was broken and something shipped wrong. The examples are
real, not hypothetical. Read this before the sensor checklist.

### 1. Check the protocol documents before inferring anything

We have ~2.5 MB of Growatt protocol documentation in `Protocols/` and a 105 KB extracted
reference in `docs/developer/protocol-v139.md`. **Search it first.** Grep the register
number. It takes seconds.

Register meanings were argued across GitHub threads for a week, twice wrongly, while the
answers were already checked in:

- Battery SOH was reported to a user as register 31218 (VPP range) and possibly
  unobtainable. It is **1096**, documented as `BMS_SOH` under "BMS information 1082-1124".
- Registers 1021/1037 were disputed for days. The doc states plainly: `1021 PactouserTotal`
  (grid import), `1037 PLocalLoad total` (house load).

A profile's `desc` string, a code comment, and a model's marketing name are **not
evidence**. The protocol document and a field scan are.

### 2. Input and holding registers overlap — always state the function code

The same address means different things in each space. This has caused three separate
errors:

| Address | Holding (FC03) | Input (FC04) |
|---|---|---|
| 43 | DTC / device type code | `Iac2` phase 2 current |
| 1083-1088 | Grid First time periods | BMS status / SOC / voltage / current / temp |
| 1100-1108 | Battery First time slots | BMS gauge and version data |

Reading input 43 and concluding a device reports no DTC — when the DTC lives in holding 43
— is a mistake that has been made *while warning someone else about the same trap*. When
you cite a register, say which space it is in.

### 3. "Read OK" and "responds" do not mean "works"

A register answering with `0` reports as **Read OK**. That is evidence the address
responds, never that the value is meaningful or that writes take effect.

Registers 1071/1091 survived three contradicting field reports because an old scan showed
"all Read OK". They accept writes and silently ignore them.

### 4. Absence of evidence needs the evidence to have been possible

Before concluding a register is empty or absent, confirm the scan actually covered it.
A v0.7.7 scan was used to prove a device reports no DTC — that scanner version never read
the legacy holding range at all. A missing range in a scan is not a missing value.

### 5. Verify the whole chain, not the half you are thinking about

Three releases needed follow-ups this week, all the same shape:

- v1.2.0: the read path consumed the block-size option correctly, and the form could never
  save it. Verified the consumer, not the producer.
- v1.3.5: fixed the option format, updated one of two fetch paths. The other raised
  `ValueError` on every poll.
- v1.4.0: removed a register so its sensor would disappear. The sensor platform recreated
  it from the profile's sensor set and it reported `0.0`.

**When you change a stored format, grep every consumer. When you remove data, check what
recreates it.**

### 6. `hasattr()` gates only work on dynamic attributes

`condition: lambda data: hasattr(data, 'x')` reads as "only if the profile provides x". It
is a no-op when `x` is a `GrowattData` dataclass field, because the field always exists
with a default — the sensor is created regardless and publishes the default, typically a
plausible-looking `0`.

It works only for attributes assigned dynamically via `setattr()` (the BMS block, for
example). 31 conditions in `sensor.py` are decorative; `tests/test_sensor_conditions.py`
enumerates them and fails if a new one appears. To exclude a sensor for real, remove it
from the profile's **sensor set** — that is the only hard filter.

### 7. Never write file content through PowerShell

Use the **Edit** and **Write** tools. `Set-Content -Encoding utf8` writes a BOM in PS 5.1,
which broke `manifest.json` and would have stopped the integration loading. Em-dashes have
been mangled into `â€"` in three separate files this way.

Prefer plain ASCII in commit messages and shell-adjacent content — hyphens rather than
em-dashes, straight quotes rather than curly. Non-ASCII in source files is fine *via the
editing tools*, never via a shell redirect.

**This matters more than it looks: `docs/` is published to GitHub Pages.** A file written
with the wrong encoding does not just look odd in an editor — it renders as visible
mojibake to every reader of the documentation site. The character is never the problem;
the encoding is. Banning em-dashes would treat the symptom and still leave curly quotes,
degree signs, arrows and accented names to break the same way.

Check before committing anything written outside the editing tools:

```bash
# BOM (breaks JSON parsing, renders as a stray glyph in Markdown)
python -c "print(open('FILE','rb').read()[:3] == b'\xef\xbb\xbf')"
```

Then grep for `â€`, `Ã¢`, `Â` — the signatures of UTF-8 read as Windows-1252.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xAHA/Growatt_ModbusTCP](https://github.com/0xAHA/Growatt_ModbusTCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
