---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project

**ALYNXDJ** — an LSDJ-inspired music tracker for the **Atari Lynx**, the third
tracker in the author's family after **SMSGGDJ** (Master System / Game Gear,
`/Users/a1106632/Documents/sms_tracker`) and **GENMDDJ** (Mega Drive,
`/Users/a1106632/Documents/genmddj`). The brief is `task.txt`: port as much of the
design, control scheme, command set, and GUI feel of SMSGGDJ/GENMDDJ as makes sense,
while exploiting everything unique to the Lynx sound hardware (Mikey's 4 channels of
LFSR/polynomial-counter synthesis, per-channel 8-bit DAC access, stereo panning on
later hardware). The ROM and project name is **ALYNXDJ**.

**DESIGN.md is the contract** (with a §0 decision log — read it before design
decisions, don't re-litigate settled ones); **PLAN.md** holds the milestone
history through M48. Key settled decisions: 4 identical
tracks each playing LFSR/WAV/KIT (D1/D35), cc65 C editor + ca65 asm
driver/IRQ/render (D2), 59.9 Hz VBlank engine tick with no region split (D3),
flat RAM song block but **packed/RLE EEPROM save — 2 KB 93C86 is the whole
persistence budget** (D4/D10), per-channel timer-IRQ PCM capped at 2 voices
(D5/D6), 4×6 font on a 40×17 grid (D7). KIT and table-WAV dynamically
target the owning channel A–D; there are no fixed sample buses. LFSR patches
(including legacy stored type `$01`) persist TSP/SWP/VIB/TRM plus TBS in
save-format v6 (D15–D17/D35); WAV retains signed TSP, while KIT reuses byte
15 as D48's source RATE (`FF` 0.5×, `00`–`03` 1×–4×). VIB is a
centred ~0.47–7.49 Hz sine whose phase free-runs across notes and resets at
the transport boundary. TBS 0 is a per-note table cycle and TBS 1–F are
tick-clocked; every mode wraps `0F→00`, while `H` sets a custom loop. KIT VOL
is D36's static foreground PCM gain:
high nibble `6`–`7` full, `2`–`5` signed `>>1`, `1` signed `>>2`, and `0`
mute; its low nibble is ignored and blocked in the UI, while the DAC IRQ
remains unchanged. D19/D20 add receive-only ComLynx MIDI takeover and
clock sync: full-status MIDI channels 1–4 route to tracks A–D/instruments
00–03. The UART IRQ buffers bytes continuously and the 59.9 Hz engine tick
drains complete messages before voice processing, so redraw cannot delay MIDI
and serial receive stays live while a multi-channel chord is applied. The Pico
emits an immediate `IN24` row grant after Start/Continue, then divides USB
or serial MIDI's 24 PPQN into one pulse per following tracker row. `IN` and `IN24` arm
locally at the cued row as `WAIT`; their first row pulse starts that row and
changes the transport to `PLAY`. MIDI CC74 enters the shared `N` executor path
as a note-local LFSR taps byte; Pitch Bend enters shared `F` as a held absolute
±2-semitone target at 1/16-semitone resolution. There is no heartbeat.
For legacy Lynx-to-Lynx OUT/IN, D55 makes the single `$02` START byte the
downbeat/first-row grant; later rows use `$01`. Do not send START+ROW adjacent:
legacy IN is polled and Mikey has only a one-byte receive holding register.
The companion bridge also accepts opto-isolated 31,250-baud serial MIDI on
GP13, expands running status, and routes it through the same dispatcher as USB
MIDI. Use one MIDI source at a time. The Chipbridge RP2040-Zero build follows
the PCB exactly: ComLynx `DATA1` = GP1, MIDI RX = GP13, and ready LED = GP7;
`DATA0`/GP0 is unused. Its Lynx cable is ring-to-ring plus sleeve-to-sleeve
only, with both tips disconnected and insulated because the Lynx tip is +5 V.
The separate D56 `pico-gg-sync` image keeps that hardware fixed: SMSGGDJ OUT
enters the Type-A MIDI optocoupler on GP13, whose differential ring/bit1 and
tip/bit0 input exposes only counter state 2. It measures the one-low/three-high
marker, reconstructs an equal row grid, and emits legacy START/ROW/STOP on GP1.
Flat GG rows are exact; arbitrary swing cannot survive the one-bit collapse.
The complete unchanged path is hardware-verified as of 2026-08-27. This repo
owns the translator firmware and tests; the cross-linked Chipbridge repo owns
the PCB, GG adapter, and shared connector hardware.
KIT bank follows D38: it is always `00`–`07`, selecting KIT or viewing an old
invalid KIT value normalizes it to `00`, and only WAV may display `--`.
KIT sample banks follow D39: the one supported `PL` format uses rate ID `1`,
5,208.333 Hz signed 8-bit PCM (timer reload 191), with up to ~12.58 seconds
per u16-sized slot. The builder handles 8/16/24/32-bit integer WAV sources;
older experimental rate IDs are intentionally rejected. D40 factory
conversion peak-normalizes, then applies +12.00 dB into a tanh soft limiter
before signed 8-bit quantization; the processing is baked into the bank.
KIT RATE repeats or skips ring bytes while keeping reload 191, so 2×–4× do
not raise IRQ frequency. `Sxx` is D47's bounded KIT source-rate override:
its low two bits select 1×/2×/4×/0.5× without changing the timer, and every
note/`R` restores the instrument stride. KIT pad selection ignores instrument
TSP/RATE but still follows phrase/chain note transpose.
Command values retain their historical/save-format IDs. PHRASE steps through
all implemented letters alphabetically; TABLE exposes only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [little-scale/alynxdj](https://github.com/little-scale/alynxdj) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
