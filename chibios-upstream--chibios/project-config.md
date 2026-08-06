---
trigger: always_on
description: - Operating system modules are under `./os`.
---

# Repository Guidelines

## Project Structure & Module Organization
- Operating system modules are under `./os`.
- Core STM32 HAL sources live under `os/hal/ports/STM32`.
- XHAL sources live under `os/xhal`; generated driver interfaces and common code come from XML descriptions under `os/xhal/codegen`.
- Shared STM32 infrastructure (RCC helpers, registry, limits) sits alongside in the same tree, while board examples and demos are under `demos/`.
- Tests and validation harnesses reside in the `test*` directories; use them to verify changes before submitting.
- Configuration templates and regeneration helpers are under `tools/ftl` and `tools/updater`; generated `xhalconf.h`/`xmcuconf.h` outputs should be treated as derived files.

## Coding Style & Naming Conventions
- Follow existing style patterns: 2-space indentation for C sources, and `STM32_*` macro naming for register constants (see `hal_lld.h`/`stm32_rcc.h`).
- Keep functions `static` unless they are part of the public HAL API.
- Regenerate formatting by hand; clang-format is not used on this port—mirror the surrounding code style.
- General rule: line endings must be LF, except for externally provided files (non-ChibiOS copyright).
- In C functions, keep automatic variable declarations grouped at the start of the block and separate the declaration block from executable statements with an empty line.
- In ChibiOS, API suffix semantics are critical and must drive design decisions: no suffix = thread context, `X` = any context, `S` = system-locked context, `I` = interrupt/system-locked context. Semantic compatibility comes before implementation convenience; never place calls in fastcall/ISR paths unless their suffix contract explicitly allows it.
- Driver callbacks must follow their context contract. Generic XHAL driver callbacks are invoked from ISR context and out of system locks; do not call them from thread-context APIs, and do not invoke them while holding `osalSysLock*()`/`chSysLock*()`. If an ISR helper must wake waiters, lock only around the `I`-class wakeup/VRQ operations, unlock, then invoke the callback.
- While debugging new code, prefer enabling assertions, parameter checks, and the state checker in `chconf.h`. In ChibiOS, contract violations often halt early with those options enabled, which is desirable for catching context and state misuse quickly.
- STM32 shared IRQ handlers follow the `.inc` pattern used by SPI/USART/RTC: platform `stm32_isr.c` includes the right `.inc`, vector names and numbers belong in `stm32_isr.h`, and the corresponding `driver.mk` must export the include directory that contains the shared `.inc` files.
- For STM32 vectors shared by multiple peripherals, choose one peripheral as the primary owner and treat the others as secondaries. The shared `.inc` filename and the IRQ priority macro must name all participating peripherals, while the primary peripheral defines the overall handler structure and inclusion point.
- For primary/secondary shared vectors, keep the ISR body high level. The `.inc` should dispatch to driver service entry points, while peripheral-specific flag decoding and per-channel/per-instance detail should live inside the driver-side service functions unless the hardware has a dedicated vector and the established pattern already keeps that logic in the `.inc`.
- XML driver descriptions must remain schema-valid and include the documentation elements expected by code generation; otherwise generated Doxygen comments disappear and regeneration becomes lossy.
- After editing any codegen or board XML (`os/xhal/codegen`, `os/vfs/codegen`, `tools/ftl/xml`, etc.), validate it against the schema named in its `xsi:noNamespaceSchemaLocation` before regenerating or committing: map that `.../schema/...` path to the local file under `tools/ftl/schema/` and run `xmllint --noout --noent --schema <local.xsd> <doc.xml>`. A failure means either the XML is wrong or the schema is stale; do not loosen the schema blindly — when correcting it, take device subtypes from the ST CMSIS headers and GPIO port sets from the in-tree `stm32_registry.h`. Note `xmllint` reports only the first error per container, so re-validate after each fix.

## XHAL Architecture Notes
- XHAL stateful drivers derive from `hal_base_driver`; lifecycle, configuration selection, mutual exclusion, and registry integration should be exposed through the base-driver API when the semantics are common.
- `drvStart()` and `drvStartS()` are the official XHAL lifecycle entry points. Driver-specific `xxxStart()`/`xxxStop()` wrappers should not be reintroduced unless there is a deliberate compatibility layer outside the generated XHAL driver API.
- `drvStart(ip, config)` performs initial hardware enable and configuration from `HAL_DRV_STATE_STOP`; `drvStart(ip, NULL)` uses configuration zero. Calling `drvStart(ip, NULL)` while already `HAL_DRV_STATE_READY` is idempotent, but passing a non-`NULL` config while ready is invalid.
- Live configuration APIs such as `drvSetCfgX()` and `drvSelectCfgX()` are only valid in `HAL_DRV_STATE_READY`. Use the `config` parameter of `drvStart()` for initial configuration because hardware may be reset or clock-gated before start.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chibios-upstream/chibios](https://github.com/chibios-upstream/chibios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
