---
trigger: always_on
description: Read this first, then [`docs/IMPROVEMENT_PLAN.md`](docs/IMPROVEMENT_PLAN.md) for what
---

# AGENTS.md — handoff for the next agent

Read this first, then [`docs/IMPROVEMENT_PLAN.md`](docs/IMPROVEMENT_PLAN.md) for what
is open and [`docs/ROADMAP.md`](docs/ROADMAP.md) for what is done.

The completed plan documents live in
[`docs/history/`](docs/history/) — including `PLAN.md`, which this line used to
send you to first. They are worth reading for *why* something was built the way
it was, and they are not current instructions; the figures in them are the
figures of their own day.

**FactoryForge** is a free, open 3D factory simulator for learning PLC
programming — a replacement for Factory I/O, which is stagnant, closed to custom
parts, and €278/year. See [`docs/PRD.md`](docs/PRD.md) for the full rationale.

---

## Absolute paths on this machine

| What | Path |
|---|---|
| **Repo** | `C:\Users\masal\source\factoryforge` |
| **Godot 4.7.2 mono** (console build — use this, it prints to stdout) | `D:\Godot_v4.7.2-stable_mono_win64\Godot_v4.7.2-stable_mono_win64_console.exe` |
| Python 3.12 | `D:\Python312` (on PATH as `python`) |
| .NET SDK | `C:\Program Files\dotnet` (v10; the project targets net8.0) |
| Node-RED user dir | `C:\Users\masal\.node-red` — **contains the user's own flows, never overwrite** |
| Factory I/O install (reference only) | `F:\Program Files (x86)\Real Games\Factory IO` |

## Live hardware

| What | Detail |
|---|---|
| **S7-PLCSIM Advanced** | V6.0 Upd1, instance `test`, **TCP/IP Single Adapter, communication with `<Local>`** (the PLCSIM virtual adapter, "Ethernet 2", 192.168.0.100/24). The CPU is `192.168.0.20`, OPC UA at `opc.tcp://192.168.0.20:4840`, S7 on 102. Until 2026-09-29 this said 192.168.1.20: that is the Wi-Fi's subnet and nothing reached it. Softbus ("PLCSIM") mode works for TIA and the native driver only, and a Multiple Adapter instance on `<Default>` fails with "Interface mapping is invalid or missing" |
| CPU | 1511-1 PN, FW V2.9, TIA Portal V19 |
| PLC program | TIA project `D:\projects\Tia portal mcp\projects\FactoryForge_Sorting` (built through TiaCommander + Openness on 2026-09-29): the 19-member `FF_IO` + FB `SortingByHeight` (the first-hour program, `examples/graded/first_hour_sorting_by_height.scl`). The `Sorting.scl` v0.4 build is archived as `D:\projects\Tia portal mcp\archives\FactoryForge_Sorting_scl_v04_*.zap19` |
| Node ids | `ns=3;s="FF_IO"."ConveyorRotate"` etc. — quotes are part of the identifier |

The user has **no physical PLC**. PLCSIM Advanced is the reference target and
simulates only the S7-1500 family. Plain bundled PLCSIM has no external network
interface and cannot be used.

---

## Architecture

```
┌────────────────────────────┐          ┌──────────────────────────┐
│  SIM ENGINE                │          │  DRIVER SIDECAR (Python) │
│  Godot 4.7 / C#  (engine/) │  tag bus │  asyncua      (OPC UA)   │
│  or Python stub (harness/) │ ◄──────► │  built-in     (Modbus)   │
│                            │  WS/JSON │  mock         (tests)    │
└────────────────────────────┘          └──────────────────────────┘
```

The **tag bus** ([`docs/tag-bus.md`](docs/tag-bus.md)) is the seam and the most
important contract in the project. Two engine implementations already speak it
(`sidecar/factoryforge_sidecar/engine_stub.py`, still importable as
`harness/engine_stub.py`, and `engine/src/TagBus/TagBusServer.cs`) and the
sidecar cannot tell them apart. Keep it that way.

`kind` is always from the **controller's** point of view: `output` = PLC writes
it, `input` = simulator writes it. This trips everyone up; it is enforced.

### Layout

```
engine/          Godot 4.7 C# project
  src/TagBus/    Tag, TagTable, TagBusServer  (C# port, must match Python)
  src/Scenes/    SortingScene.cs              (port of sidecar/.../sorting_scene.py)
  src/View/      SceneView.cs, OrbitCamera.cs (reads state only, never writes)
harness/         aliases: engine_stub.py and scene.py *are* the sidecar's modules
sidecar/         Python package: bus client, drivers, minimal Modbus server,
                 the Python engine reference (engine_stub.py), the headless
                 sorting scene (sorting_scene.py) and the grader (grading/)
examples/tia/    Sorting.scl, FF_IO DB spec, setup walkthrough
examples/nodered/ flow that replaces the PLC entirely
tools/           drive_engine.py (parity check), drv_trace.py (driver tracing)
tests/           the pytest suite; no Siemens software and no GPU required
```

---

## Commands

```bash
cd C:/Users/masal/source/factoryforge

# The whole test plan: build, pytest, engine self-tests, determinism, the
# engine/sidecar seam, robustness. Needs no PLC. See docs/TEST_PLAN.md.
python tools/test_plan.py            # add --gui for the display-dependent check

# The Python suite. How many pass is whatever it prints: this comment used to
# carry the number, and still said 41 long after the suite outgrew it. A6b in
# test_plan.py now fails if a test count is written into this file.
# -m "not graded" skips the slow grader tests; -m graded runs only them.
python -m pytest -q

# Engine build
cd engine && dotnet build

# Drive a RUNNING engine from a real driver. This is the one to use with the
# 3D engine; `demo` starts its own Python scene on the bus port, so against a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agentthink/simplant](https://github.com/agentthink/simplant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
