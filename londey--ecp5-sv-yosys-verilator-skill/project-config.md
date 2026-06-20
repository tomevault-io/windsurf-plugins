---
trigger: always_on
description: >
---


# ECP5 SystemVerilog: Yosys + Verilator Compatibility

## Core Principle: Always Instantiate Hard Resources Explicitly

For DP16KD (56 on ECP5-25K) and MULT18X18D (28 on ECP5-25K), **do not rely on Yosys inference**. Inference is incomplete and unreliable: ALU54B features are not inferred at all, MULT9X9D (two 9x9 mults per DSP tile) is not supported by Yosys/nextpnr, and BRAM inference fails for wider-than-18-bit or multi-port patterns. Explicit instantiation gives you exact resource consumption, correct pipeline register placement, and predictable timing closure.

---

## ECP5-25K Hard IP Budget (CABGA256/CABGA381)

| Resource         | Count | Notes                                                     |
|-----------------|-------|-----------------------------------------------------------|
| DP16KD          | 56    | 18Kbit true dual-port EBR; also configurable as PDPW16KD  |
| MULT18X18D      | 28    | 18x18 signed multiplier; shares tile with ALU54B          |
| ALU54B          | 14    | 54-bit accumulator; paired 2:1 with MULT18X18D            |
| EHXPLLL         | 2     | Integer-N PLL (2 on -25K, 4 on larger variants)           |
| DCCA            | 56    | Clock distribution / CE gates                             |
| CLKDIVF         | 4     | Integer clock dividers                                    |
| TRELLIS_ECLKBUF | 8     | Edge-clock buffers for DDR clocking                       |
| ECLKSYNCB       | 10    | Edge-clock synchronizer/gate                              |
| OSCG            | 1     | Internal ring oscillator (~155 MHz / DIV)                 |
| JTAGG           | 1     | User access to JTAG port                                  |
| USRMCLK         | 1     | User access to SPI flash MCLK                             |
| GSR             | 1     | Global Set/Reset network driver                           |
| IOLOGIC         | 128   | High-speed I/O registers (DDR, gearbox)                   |
| SIOLOGIC        | 69    | Single-data-rate I/O registers                            |

---

## The Golden Pattern: Macro-Guarded Instantiation

```systemverilog
// Build system: -D SYNTHESIS for Yosys; omit for Verilator
`ifdef SYNTHESIS
  EHXPLLL #(...) pll_inst (...);
`else
  pll_sim_model pll_inst (...);
`endif
```

---

## Block RAM: DP16KD and PDPW16KD

### Two EBR Configurations

The same physical 18Kbit EBR block is configured as one of two named primitives:

**`DP16KD`** — True Dual-Port (TDP). Both ports read/write independently, each with its own clock. Port width up to 18 bits (16 data + 2 parity). Yosys internal name `$__ECP5_DP16KD`.

**`PDPW16KD`** — Pseudo Dual-Port Wide (SDP mode). Port A is write-only at **36 bits** (32 data + 4 parity). Port B is read-only, configurable at 9, 18, or 36 bits — asymmetric reads allowed. This is the only way to achieve 32-bit data width from a single EBR. Yosys internal name `$__ECP5_PDPW16KD`.

These are mutually exclusive configurations of the same hardware — you cannot mix them.

### DP16KD Aspect Ratios

| DATA_WIDTH | Addr bits active | Depth    | Data | Parity |
|-----------|-----------------|----------|------|--------|
| 1         | ADA13:ADA0      | 16384x1  | 1    | 0      |
| 2         | ADA13:ADA1      | 8192x2   | 2    | 0      |
| 4         | ADA13:ADA2      | 4096x4   | 4    | 0      |
| 9         | ADA13:ADA3      | 2048x8   | 8    | 1      |
| 18        | ADA13:ADA4      | 1024x16  | 16   | 2      |
| 36        | ADA13:ADA5      | 512x32   | 32   | 4      |

Port A and Port B can have **different widths** (asymmetric). The lower address pins not used by the selected width must be tied to `1'b0`.

### DP16KD Instantiation (1K x 16-bit TDP)

```systemverilog
`ifdef SYNTHESIS
DP16KD #(
  .DATA_WIDTH_A(18),      // 16 data + 2 parity
  .DATA_WIDTH_B(18),
  .REGMODE_A("NOREG"),    // "OUTREG" adds 1 cycle latency, improves Fmax
  .REGMODE_B("NOREG"),
  .RESETMODE("SYNC"),
  .ASYNC_RESET_RELEASE("SYNC"),
  .WRITEMODE_A("NORMAL"), // "NORMAL"=read-before-write, "WRITETHROUGH"=pass-through
  .WRITEMODE_B("NORMAL"),
  .GSR("ENABLED"),
  .INIT_DATA("STATIC"),
  .CSDECODE_A("0b000"),   // chip select decode; 0b000 = always selected
  .CSDECODE_B("0b000")
) bram_i (
  // Port A
  .CLKA(clk_a), .CEA(1'b1), .OCEA(1'b1), .RSTA(rst),
  .WEA(we_a), .CSA0(1'b0), .CSA1(1'b0), .CSA2(1'b0),
  // For DATA_WIDTH=18: active addr is ADA13:ADA4; tie ADA3:ADA0 to 0
  .ADA13(addr_a[9]),.ADA12(addr_a[8]),.ADA11(addr_a[7]),.ADA10(addr_a[6]),
  .ADA9(addr_a[5]),.ADA8(addr_a[4]),.ADA7(addr_a[3]),.ADA6(addr_a[2]),
  .ADA5(addr_a[1]),.ADA4(addr_a[0]),
  .ADA3(1'b0),.ADA2(1'b0),.ADA1(1'b0),.ADA0(1'b0),
  // DIA17:DIA16 = parity (tie to 0 if unused)
  .DIA17(1'b0),.DIA16(1'b0),
  .DIA15(din_a[15]),.DIA14(din_a[14]),.DIA13(din_a[13]),.DIA12(din_a[12]),
  .DIA11(din_a[11]),.DIA10(din_a[10]),.DIA9(din_a[9]),.DIA8(din_a[8]),
  .DIA7(din_a[7]),.DIA6(din_a[6]),.DIA5(din_a[5]),.DIA4(din_a[4]),
  .DIA3(din_a[3]),.DIA2(din_a[2]),.DIA1(din_a[1]),.DIA0(din_a[0]),
  .DOA17(),.DOA16(),
  .DOA15(dout_a[15]),.DOA14(dout_a[14]),.DOA13(dout_a[13]),.DOA12(dout_a[12]),
  .DOA11(dout_a[11]),.DOA10(dout_a[10]),.DOA9(dout_a[9]),.DOA8(dout_a[8]),
  .DOA7(dout_a[7]),.DOA6(dout_a[6]),.DOA5(dout_a[5]),.DOA4(dout_a[4]),
  .DOA3(dout_a[3]),.DOA2(dout_a[2]),.DOA1(dout_a[1]),.DOA0(dout_a[0]),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [londey/ecp5-sv-yosys-verilator-skill](https://github.com/londey/ecp5-sv-yosys-verilator-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
