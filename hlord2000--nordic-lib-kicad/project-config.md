---
trigger: always_on
description: This document summarizes the current symbol layout conventions in `symbols/*.kicad_sym`.
---

# Symbol Arrangement Guide

This document summarizes the current symbol layout conventions in `symbols/*.kicad_sym`.
It is based on direct parsing of the KiCad S-expression files, not on `kiutils`.

## Reading Symbol Layout

- Pin side is encoded by pin angle: `0` means left edge, `180` means right edge, `270` means top edge, and `90` means bottom edge.
- Left and right pins are read top-to-bottom by descending `Y`. Top and bottom pins are read left-to-right by ascending `X`.
- KiCad nested symbol names such as `_0_0`, `_1_0`, `_2_0` are part of the raw symbol storage. `_0_0` commonly holds pins shared by the visible unit; `_1_0`, `_2_0`, etc. hold unit-specific pins. `_1_1` and similar blocks are usually graphics or alternate body blocks.
- Symbols that use `(extends "...")` inherit the base symbol geometry and usually only change properties such as value, description, footprint, filters, or datasheet.

## General Arrangement

### SoC and SiP Symbols

- The main I/O edge is usually the left edge. It starts with clock/crystal pins (`XC1`, `XC2`, sometimes `XL1`, `XL2` as pin names or alternates), then GPIO groups in port order.
- USB-capable SoCs place `D+` and `D-` at the upper left before or near the crystal pins.
- NFC-capable SoCs keep the NFC pair embedded in the GPIO sequence:
  - nRF52: `P0.09`/`P0.10` or `P0.09/NFC1`/`P0.10/NFC2` on the left.
  - nRF53: `P0.02`/`P0.03` or `P0.02/NFC1`/`P0.03/NFC2`.
  - nRF54L and BL/BM15x modules: `P1.02`/`P1.03` with `NFC1`/`NFC2` alternates.
  - BM20x modules: `P2.10/NFC1` and `P2.11/NFC2` at the lower right.
- RF antenna pins are normally on the right. For most SoCs `ANT` is at the right-center area, commonly around `y=0`. nRF9 SiPs place `ANT` near the upper right and also expose `GPS` on the right. RF FEM/module symbols may use `TRX`, `ANT1`, `ANT2`, or `OUT_ANT` instead.
- Debug/reset pins are usually at the lower right in this order from top to bottom: `~{RESET}`, `SWDCLK`, `SWDIO`. Exceptions are modules whose castellated or LGA edge grouping puts debug on the left, and nRF54LS05A where `P1.30` and `P1.29` carry `SWDCLK` and `SWDIO` as alternates.

### Power, Ground, and Regulator Pins

- Power input pins are usually on the top edge. Grounds are usually on the bottom edge.
- Duplicate package supply pins are hidden and stacked at the same coordinate as a visible representative supply pin.
- Duplicate grounds, exposed pads, shield grounds, and analog grounds are hidden and stacked at the visible bottom ground pin unless a distinct net is required.
- nRF52 decoupling/regulator pins use the right edge for `DEC*` pins and the top edge for `DCC`. USB/high-voltage variants add `VBUS`, `VDDH`, `DECUSB`, and sometimes `DCCH`.
- nRF53 places several regulator/decoupling pins interleaved with I/O: `DCC`, `DECR`, `DECRF`, `DCCD`, and `DECD` on the left side region; `DECA`, `DECN`, `DCCH`, and `DECUSB` on the right side region.
- nRF54L places decoupling/regulator pins at the top of the right edge, generally `DECA`, `DECRF`, `DECD`, `DCC`, then `ANT`, then reset/SWD.
- nRF54LV adds charge-pump/low-voltage pins on the right: `DECA`, `DECB`, `DECRF`, `DECD`, `DCC`, `CFLYH`, `CFLYL`, then `ANT` and reset/SWD.
- nPM PMIC symbols place source supplies on the top or upper left, regulator outputs and switch nodes on the right, configuration/control pins on the left, and grounds on the bottom.
- nRF7 Wi-Fi companion symbols place digital host/control pins on the left, RF/buck/analog supply pins on the right, supplies on the top, and ground on the bottom.

### Alternate Modes

- Primary pin names should remain the package or port names when that is what the existing symbol uses. Peripheral functions should be expressed as KiCad `alternate` entries.
- Current alternate-mode families include:
  - nRF52: `AIN`, `XL`, `NFC`, and trace alternates on larger parts.
  - nRF53: `AIN`, `XL`, `NFC`, `TWI`, `QSPI`, high-speed SPI, and trace alternates, mainly on `nRF5340-QKXX`.
  - nRF54L and nRF54 modules: `AIN`, `ASI`, `ASO`, `RADIO`, `QSPI`, `SPIM`, `SPIS`, `UARTE`, `TRACE`, `SWO`, and package-specific functions.
  - nRF54LV: `AIN`, `TRACE`, `ASI`, `ASO`, `FLPR`, `RADIO`, `QSPI`, `SWO`, and crystal alternates depending on package.
  - nRF7: QSPI pins expose SPI alternate names.
  - nPM2100: `VOUTLDO` has `LSOUT` as an alternate.

## Per-Library Symbol Summary

### `nordic-lib-kicad-npm.kicad_sym`

- `nPM1100-CAXX`: PMIC/charger BGA. Left: USB `D+`/`D-`, NTC/config/charge pins. Right: `DEC`, `VOUTBSET0`, `VOUTBSET1`, `VOUTB`, `SW`, mode/ISET/shutdown pins. Top: `VBAT`, `VBUS`, `VSYS`. Bottom: stacked `VSS`.
- `nPM1100-QDXX`: Same arrangement as `nPM1100-CAXX`, with QFN grounding split as visible `AVSS` and `PVSS`.
- `nPM1300-CAXX`: Left: `VBUS`, USB-C CC pins, LEDs, GPIO/I2C, `VSET1`, `VSET2`, `SHPHLD`. Right: `VBUSOUT`, buck `VOUT1/SW1/PVSS1`, buck `VOUT2/SW2/PVSS2`, LDO/load-switch paths, `VBAT`, `NTC`. Top: `VDDIO`, `PVDD`, `VSYS`. Bottom: stacked `VSS`.
- `nPM1300-QEXX`: Same logical grouping as `nPM1300-CAXX`; bottom ground is visible `AVSS`.
- `nPM2100-CAXX`: Left: `VBAT`, `SW`, `VSET`, I2C, GPIO. Right: `VOUT`, `VOUTLDO`, `VINT`, `SHPHLD`, `PG/RESET`, `SYSGDEN`. Bottom: `PVSS` with hidden analog grounds stacked. `VOUTLDO` has `LSOUT` alternate.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hlord2000/nordic-lib-kicad](https://github.com/hlord2000/nordic-lib-kicad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
