---
trigger: always_on
description: DIY multi-sensor environmental monitor for indoor spaces. Measures air quality and presence. Mounts on a standard wall-recessed electrical box (60 mm screw pitch). Powered from 24 V DC. Integrates with Home Assistant via ESPHome.
---

# OAS — Open Ambient Sensor

DIY multi-sensor environmental monitor for indoor spaces. Measures air quality and presence. Mounts on a standard wall-recessed electrical box (60 mm screw pitch). Powered from 24 V DC. Integrates with Home Assistant via ESPHome.

---

## Design philosophy

The open-source DIY space offers many indoor air quality projects. Most optimize aggressively for cost. **OAS holds two priorities non-negotiable**, even at the cost of a higher per-unit BOM:

1. **Measurement quality.** Sensirion SEN66 (calibrated combo NDIR / laser PM / MOX VOC + NOx / SHT) over cheap MOX-only alternatives. LD2410 mmWave (stillness detection) over PIR. PCB layout enforces thermal separation between heat sources and sensor inlets.

2. **Aesthetic acceptability.** The device lives in inhabited rooms. A commercial-grade injection-molded enclosure (SZOMK AK-N-94, white perforated ABS) replaces the typical 3D-printed box. No protruding modules, no exposed wiring, no fan whine.

OAS sits between bargain DIY kits and premium commercial sensors; both compromises are rejected. When evaluating any future component or design change, it must clear both bars — flag anything that breaks either pillar.

---

## Lessons learned (v0.40 audit-16)

Concrete, mandatory practices distilled from the v0.40 JLCPCB rejection saga + the 78-agent paranoid sweep + the audit-16 follow-up. Every item below is a real failure mode the project hit and recovered from.

### 1. NEVER write custom footprint stubs by approximation

Always parse-and-emit from the KiCad stock library verbatim via `_emit_stock_lib_footprint(src_path=…, lib_nickname=…)`. The pattern: stub generators are ~15-line wrappers that delegate to the helper — they MUST NOT contain hand-coded pad coordinates / sizes / silk geometry.

Root cause behind the v0.40 JLCPCB rejection (U1 LM2596S TO-263-5 emitted a 90°-rotated, miniaturized land pattern) AND ~24 SMD passive deviations caught later (0402 / 0603 / 0805 caps, 0603 resistors, SMA / SMB / SOD-323 diodes, 5×5 inductors, 2920 polyfuse, SOT-23). Q1 SOT-23 was particularly bad: custom "⊥" pad pattern vs canonical stock "E" pattern → the AO3401A would have been physically rotated 90° relative to the pads after JLCPCB's tape-feeder orientation lookup, swapping G/S/D nets and silently destroying reverse-polarity protection on first power-up.

The audit-16 sweep eliminated EVERY hand-coded pad geometry. The current header inventory of `oas.kicad_pcb` is listed under "Deviation budget" further down — every entry is either canonical KiCad stock (`<lib>:<name>`) or a project-local mechanical / NPTH / LED reference.

### 2. Footprint property string MUST be canonical `Lib:Name`

Never bare names ("PinHeader_1x06_P2.54mm_Vertical" without `Connector_PinHeader_2.54mm:` prefix). Never custom names ("TO-263-5_LM2596" instead of stock `Package_TO_SOT_SMD:TO-263-5_TabPin3`).

The `(footprint "Lib:Name"` header inside `oas.kicad_pcb` is what JLCPCB's matchers AND KiCad's `lib_footprint_mismatch` ERC both resolve against. Audit-16 caught FOUR additional canonical-name defects that audit-15 (which focused on pad geometry) had missed: U1 / U2 non-canonical headers, J2 missing lib prefix, ZT1..ZT4 missing `oas:` prefix.

### 3. NO `rule_severities` suppression — fix the cause, never paper over the warning

The audit-15 fix for Q1's `lib_footprint_mismatch` was wrong-class: it suppressed the warning via `rule_severities: {"lib_footprint_mismatch": "ignore"}` because the schematic used `Device:Q_PMOS` (letter pin numbers D/G/S) paired with a `pin_name_map` remapping stock SOT-23 pads "1"/"2"/"3" → "G"/"S"/"D".

The right fix (audit-16) was structural: create project-local `OAS:Q_PMOS_GDS` schematic symbol with NUMERIC pin numbers 1/2/3 (pin NAMES still G/S/D for readability) so the netlist binds Q1.1/Q1.2/Q1.3 canonically to stock SOT-23 pads "1"/"2"/"3" with NO remap. `gen_pro()` `rule_severities` now empty `{}` — no special-case overrides anywhere in the project.

### 4. JLCPCB DFM machine check ≠ JLCPCB human review

The v0.40 board held a stable JLCPCB DFM result of **0 DANGER / 193 W** (warnings only, fully manufacturable) — but the DFM machine did NOT catch the U1 / U2 broken footprint geometry. JLCPCB's human parts-placement reviewer caught them at the manual stage AFTER payment, and rejected the order.

Plan defensively: verbatim stock library is the only way to be safe at BOTH check layers. DFM PASS is necessary but not sufficient.

### 5. JLCPCB Assembly Order XLS pre-payment cross-check is mandatory

"Smart-match by LCSC SKU" can silently substitute wrong parts even with an explicit LCSC# specified. The v0.40 order caught THREE catastrophic near-misses at this step:
- **R3 (30.9 k 1%)**: `lcsc-mapping.csv` had `C23116` but JLCPCB's database mapped that SKU to `0603WAF8060T5E = 806 Ω` (three orders of magnitude wrong). At 806 Ω in the TPS62933 FB divider, Vout target ≈ 100 V → buck saturates at Vin → 22 V on the 3V3 rail → ESP32-C6 + SEN66 destroyed instantly. Correct SKU is `C23022`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hubertciebiada/open-ambient-sensor](https://github.com/hubertciebiada/open-ambient-sensor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
