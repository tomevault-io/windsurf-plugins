---
trigger: always_on
description: validates `unit_of_measurement` against `device_class`, and `Â°C` is not one of
---

# CLAUDE.md

Guidance for AI agents working in this repository.

## What this project is

A CODESYS 3.5 (SP21) home-automation PLC project for WAGO PFC100/200 controllers.
Critical logic runs in the PLC; MQTT carries events out to a broker and commands
back in. `README.md` is the user-facing entry point; `docs/SoftwareArchitecture.md`
explains the layering. Function blocks are documented one per page under
`docs/FunctionBlocks/`.

| Path | What it is |
|:--|:--|
| `src/HomeAutomation.project` | The real project. A **binary** CODESYS file. |
| `src/Exports/PLCopen.xml` | Generated PLCopen XML export of all IEC content. Readable. |
| `src/Libraries/` | Vendored `.library` dependencies (MQTT, OSCAT, PRO_JSON). |
| `tools/ai/` | Headless CODESYS driver. |
| `.ai/` | Gitignored scratch: candidate blocks, sandbox copy, compiler reports. |

This project is the **reference**. Real buildings run separate *installation*
projects that reuse its function blocks but own their own logic, I/O and device
tree. `sync-implementation-project` is how one of those catches up; nothing in
this repository ever downloads to a building's PLC.

## The core constraint

`src/HomeAutomation.project` is a binary. It cannot be read, diffed or edited as
text. The way in and out is **PLCopen XML**, which CODESYS both exports and
imports — and since CODESYS ships a ScriptEngine and runs headless with `--noUI`,
the real compiler can check the result rather than you having to guess from the
XML.

## Skills — use them, don't improvise

| Task | Skill |
|:--|:--|
| Add or change a function block, refactor ST, check that something compiles, re-export the PLCopen XML, inspect the project structure | **`codesys-loop`** |
| Rename an object or a variable, with every reference to it | **`codesys-loop`** (the `rename` task) |
| Regenerate or check the generated regions of `docs/FunctionBlocks/*.md` | **`update-fb-docs`** |
| Check whether the logic actually *works* — lights, pushbuttons, covers, HVAC — on a real PLC | **`test-plc-logic`** |
| Bring a real building's installation project up to this project's function blocks, or check whether it is still version-compatible | **`sync-implementation-project`** |
| Find out why a PLC stopped — a dead building, an exception, a crash, a restart nobody explained, or what a red LED on a cabinet means | **`troubleshoot-plc-exception`** |

**Naming is decided, not open.** Objects are `PREFIX_` + SCREAMING_SNAKE (`FB_`,
`F_`, `PRG_`, `I_`, `E_`, `ST_`, `A_`, `GVL_`); variables are a type prefix plus
PascalCase, with `b` for BOOL and `by` for BYTE per the CODESYS guide. The one
exemption is the `MQTT_DISCOVERY_*` structs, whose member names are published as
Home Assistant discovery keys. Do not re-derive any of this from the surrounding
code: [`docs/CodingStyle.md`](docs/CodingStyle.md) is the full version and
[`src/CLAUDE.md`](src/CLAUDE.md) the working summary, loaded automatically when
you edit the project.

**A docs page describes the present.** It says what the code does now — never what
it used to do, what changed, or what an earlier version published. The history is
already in the commit message and the pull request, dated; on the page it only
costs the reader their ability to tell what is still true.
[`docs/CLAUDE.md`](docs/CLAUDE.md) is the rule in full, and loads automatically
when you edit under `docs/`.

Two things worth knowing before you start, both covered in detail by
`codesys-loop`:

- Run `./tools/ai/codesys.ps1 doctor` to check the toolchain.

  | Dependency | Needed by | Notes |
  |:--|:--|:--|
  | CODESYS 3.5 SP21 (3.5.21.30) + PFC200 SL package | everything | No licence needed for `--noUI` scripting. |
  | WAGO Device Support Package 2.0.8.9 | the build itself, since the DALI block landed | Supplies `WagoAppDALI`. **Not vendored** — WAGO's licence forbids redistributing it, so it is installed per machine: `docs/WagoPfcPrep.md#installing-the-wago-libraries-dali`. The version is pinned to the CODESYS patch level. |
  | Python 3.12 | `update-fb-docs` only | **Installed** at `C:\Program Files\Python312-arm64`. Invoke it as **`py`**. |
  | mosquitto clients | `Mqtt-Snapshot.ps1` and `check_mqtt_discovery.py` — the runtime checks | Not on `PATH`; the tooling also looks in `C:\Program Files\mosquitto`. |

  **Trust `doctor`, not a bare `python3`.** Windows ships an App Execution Alias
  stub at `WindowsApps\python3.exe` that prints *"Python was not found; run
  without arguments to install from the Microsoft Store"* and exits non-zero
  **even when Python is installed** — the python.org build provides `python.exe`
  and the `py` launcher but no `python3.exe`. `doctor` already skips anything
  under `WindowsApps`, so it reports the truth; a bare `python3 --version` does
  not. That stub is why this file used to claim Python was missing when it had
  been installed for days, and why a whole branch of docs got hand-maintained for
  no reason.
- A successful build does **not** always mean your code was checked — an
  unreferenced POU is never compiled. Read the `harness` section of the verify
  report, not just the result line.

Anything that writes `src/HomeAutomation.project`, `src/Exports/PLCopen.xml` or

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MichielVanwelsenaere/HomeAutomation.CoDeSys3](https://github.com/MichielVanwelsenaere/HomeAutomation.CoDeSys3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
