---
trigger: always_on
description: This file is required reading before reviewing or changing IOsonata.
---

# IOsonata Repository Work Guide

This file is required reading before reviewing or changing IOsonata.

IOsonata is not a collection of independent drivers. Its design is visible only
when the base classes, generic implementation, MCU ports, and application
examples are read together. Do not infer architecture from one file, one target,
or one MCU vendor.

## Required preflight

Before changing a subsystem:

1. Read its public generic header.
2. Read its complete generic implementation.
3. Search for every target implementation and every public symbol definition.
4. Read every target implementation affected by the change.
5. For shared behavior, read representative implementations from every MCU
   architecture or vendor family where the subsystem exists.
6. Read at least one working example and the relevant tests.
7. Check recent merged work touching the same subsystem.
8. State which existing pattern the change follows.

For changes to `Device`, `DeviceIntrf`, common configuration structures, return
semantics, lifecycle, interrupt behavior, DMA behavior, or asynchronous behavior,
a Nordic-only review is never sufficient. At minimum, search and inspect the
available implementations across:

- Nordic Arm ports;
- ST STM32 ports;
- Renesas Arm and RISC-V ports;
- Microchip SAM ports;
- NXP LPC ports;
- RISC-V Espressif ports;
- host ports when the changed API is implemented there.

Do not change a shared base API without explicit authorization.

Hardware build and test output from the maintainer is authoritative.

## Port status is part of the review

Not every port has the same maturity or supported operating modes. During the
preflight, classify each relevant port as one of:

- current and complete for the behavior being studied;
- intentionally limited, such as polling-only or master-only;
- incomplete, with stubs or TODO paths;
- stale, using older aliases, fields, or copied implementation code.

A limited or stale port is not the architectural reference. It still matters for
source compatibility and for understanding what a shared change may break.
Never generalize framework behavior from an unfinished port, and never ignore an
unfinished port when changing a common header or function signature.

## Core object model

### DeviceIntrf: the transport root

`DeviceIntrf` represents a means of transferring data. It is not limited to a
physical bus.

Physical implementations include:

- UART;
- I2C;
- SPI, QSPI, and OSPI;
- USB;
- network transports.

Software and internal implementations can also be `DeviceIntrf` objects:

- `Slip`, which layers framing over another `DeviceIntrf`;
- `BtIntrf`, which presents a GATT service as a serial-style interface;
- internal controller access paths such as `CracenIntrf` and `NvmIntrf`.

The generic transfer shape is:

```text
StartRx(selector) -> RxData -> StopRx
StartTx(selector) -> TxData -> StopTx
Read              -> command/address phase -> receive phase
Write             -> command/address phase -> transmit phase
```

`StartRx` and `StartTx` open a transfer and pass a target-defined selector. They
are not universally physical bus START conditions. `StopRx` and `StopTx` close
the corresponding transfer and release its interface-level serialization.

The selector meaning depends on the interface:

- SPI uses it as a zero-based chip-select index;
- I2C uses it as the 7-bit device address;
- CRACEN uses it to select the PKE register block, PKE operand memory,
  CryptoMaster, or RNG engine;
- UART normally ignores it because the stream endpoint is already selected by
  the UART instance;
- another interface may define another selector meaning.

The start and stop hooks also perform interface-specific work. SPI may assert and
deassert chip select. I2C may generate bus START, repeated START, and STOP
conditions. CRACEN selects an engine or memory window and clears the transfer
selection when the operation closes. UART implementations commonly use no-op
start and stop hooks.

`DevIntrf_t::bBusy` protects one transfer. The framework acquires it in
`DeviceIntrfStartRx()` or `DeviceIntrfStartTx()` and releases it in the matching
stop helper. An implementation hook must not independently take or clear that
flag.

`DevIntrf_t::EnCnt` is the shared-interface enable reference count. The physical
interface is enabled on the 0 to 1 transition and disabled on the last release.
Older ports may initialize or use this field differently; inspect them before
changing lifecycle behavior.

### Polling, interrupt, and DMA use

Execution mode is selected case by case. It is not a fixed property of UART,
I2C, SPI, master mode, or slave mode.

UART is normally interrupt driven because its main use is continuous streaming.
RX data can arrive at any time, and blocking until an entire stream is complete
is not useful. UART ports therefore commonly use interrupt-driven CFIFO
producers and consumers, with optional DMA for larger TX bursts.

I2C and SPI master operations are commonly short, bounded transactions. Polling
is often the simplest and fastest implementation because interrupt setup,
context switching, and completion handling can cost more than the transfer.
Interrupt or DMA operation is still appropriate for long transfers, strict CPU

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IOsonata/IOsonata](https://github.com/IOsonata/IOsonata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
