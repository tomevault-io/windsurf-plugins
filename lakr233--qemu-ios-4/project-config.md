---
trigger: always_on
description: - Give every large artifact under `/private/tmp` an explicit owner and lifetime. Remove failed or disposable run directories immediately after extracting the required evidence, perform a post-run cleanup, and never accumulate multiple full NAND images without a checked free-space budget and an explicit reason to retain each one.
---

# iPhone machine-model invariants

- Give every large artifact under `/private/tmp` an explicit owner and lifetime. Remove failed or disposable run directories immediately after extracting the required evidence, perform a post-run cleanup, and never accumulate multiple full NAND images without a checked free-space budget and an explicit reason to retain each one.
- Model the S5L8900 reset VROM as an alias of the single physical boot-ROM region. Disable that alias at the SYSIC VROM power transition so RAM becomes visible at address zero; never represent the transition with duplicate ROM or RAM contents.
- On current upstream QEMU, give migration-registered RAM/ROM either a `DeviceState` owner or no owner; a `MachineState` may own aliases and MMIO regions but is not a valid RAM/ROM migration owner.
- Do not substitute QEMU's PL190 model for the S5L8900 PL192: the consumed contract includes part ID `0x192`, 32 vector/priority slots, `VICSWPRIORITYMASK` at `0x24`, and acknowledge/complete through `VICADDRESS` at `0xf00`.
- Chain S5L8900 VIC1 into VIC0 instead of OR-ing their CPU outputs: XNU acknowledges only VIC0, expects the downstream vector to propagate through its `VICADDRESS`, and completes a VIC1 interrupt child-to-root.
- When MTTCG lets independent device producers update one daisy-chained VIC, serialize the complete chain's input level, priority/acknowledge state, and CPU output publication under one shared lock. Protecting only each controller or atomically changing one input bit still permits a stale cross-controller recomputation to leave an owner-cleared interrupt asserted. Keep single-threaded TCG on its already serialized lock-free path so a correctness guard for an experimental mode does not tax every interrupt in the stable mode.
- Anchor the S5L8900 RTC counter to its consumed 12 MHz timebase. Keep per-timer divider/source behavior conservative until a live producer/consumer pair proves it; OpeniBoot's event timer requires a 120,000-count interval to produce 100 Hz.
- Accept a USB-over-IP adapter only after the pinned `pymobiledevice3.irecv.IRecv` object has enumerated through a real loopback socket and exercised configuration; parser-only tests do not prove the runtime monkeypatch or PyUSB-shaped interface.
- For deterministic early-Guest debugging, hold the loaded kernel before its boot command, attach and arm raw-address breakpoints while the CPU is stopped, then release the boot command; attaching after handoff races fast driver startup.
- Preserve the timer mode encoded by configuration bit 4: 8C148 uses mode 2 (configuration `0x1150`) for rearmed one-shot deadlines, while OpeniBoot's mode 0 (`0x7040`) event timer auto-reloads. Treating both as periodic turns a minimum deadline into an interrupt storm.
- Keep a development NAND persistent across ordinary prepare and boot targets; expose full erasure as a separate explicit target so a restart cannot silently destroy a completed format or restore.
- When detaching the minimal GDB client from QEMU's process-aware ARM target, address process 1 explicitly with `D;1`; an unqualified `D` can return `E22` and leave the VM paused after an otherwise-successful authenticated memory patch.
- Complete each DWC2 endpoint transaction by updating remaining bytes, packet count, and setup count before raising its interrupt; a Guest may reject an otherwise-correct DMA payload when any consumed transfer counter still describes an unreceived packet.
- Publish PMU cable-presence status from an explicit external connection property, and keep the consumed presence bit read-only even when adjacent charger registers remain stored; otherwise Guest writes or a zero-filled reset image can spuriously enter a physical power-off path.
- Bind N82 multitouch from the selected 8C148 DeviceTree and firmware personality: Zephyr2/Z2F52 is on SPI1 with chip select pad 24 pin 0, power pad 7 pin 1, reset pad 6 pin 6, and attention GPIO 155; do not substitute the iPhone 2G Z1 or a later N1 protocol identity.
- In the S5L8900 device-mode USB controller, keep command bits self-clearing and derive `DAINT`, global endpoint status, and IRQ level from endpoint state plus masks; do not migrate duplicate summaries that can disagree with their endpoint owner.
- Treat CPU count as a per-machine hardware contract: iPhone1,2 exposes exactly one ARM1176 core and rejects `-smp 2`; add later iPhone generations as separate machine types instead of widening this board.
- Treat `IOSU ENUMERATE` as host-adapter discovery, not restore evidence. Prove the transport with one combined witness in which pinned `IRecv` requests traverse Guest-programmed endpoint DMA buffers and complete through the same endpoint interrupt path; neither the Host adapter nor a register-only test is sufficient alone.
- Validate an S5L8900 LCD scanout with an actual fixed-size screendump and sentinel pixels across the Guest RAM/alias path; register round-trips alone do not prove window selection, stride, byte order, or color conversion.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lakr233/qemu-ios-4](https://github.com/Lakr233/qemu-ios-4) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
