---
trigger: always_on
description: Source of truth for agent instructions in this repo. Read this before any work; read the nested AGENTS.md nearest the code you're editing (list below).
---

# AGENTS.md

Source of truth for agent instructions in this repo. Read this before any work; read the nested AGENTS.md nearest the code you're editing (list below).

## Hard Constraints

- **`libs/prodigy-oaa-protocol/proto/` is hands-off** — community submodule ([open-android-auto](https://github.com/mrmees/open-android-auto)). Note needed proto changes; never edit them here.
- **`proto/api/` is FROZEN additive-only** (since `875feaf`): field numbers never reused, messages never renamed, semantics never silently changed. New capability = new field + capability flag.
- **Wireless-only AA.** No USB/libusb transport — BT discovery → WiFi AP → TCP.
- **Qt 6.8 system packages.** WSL2 Debian Trixie dev environment = Pi target; no CMAKE_PREFIX_PATH, no vendored Qt.
- **HF/AG roles:** the Pi is the HFP Hands-Free (0x111e); the phone is the Audio Gateway. If you're registering profile 0x111f on the Pi, stop.
- **No ofono, no `provide-ofono`** — telephony goes through `org.pipewire.Telephony` directly.
- **External API rails:** the API binds providers/services, never EventBus topics, D-Bus paths, or AA protocol internals; all mutation through ActionRegistry or explicit invokables; additive proto only; the JS shim gets no capability the public API lacks.
- **Frozen numerics:** `ICallStateProvider` values (`Idle=0, Ringing=1, Active=2`), overlay z-bands (1000/2000/3000/3500/4000), `DashboardContributionKind` order, YAML placement field names. Append, never renumber.

## Overview

Clean-room open-source rebuild of OpenAuto Pro (BlueWave Studio, defunct): a Raspberry Pi 4 wireless-only Android Auto head unit. Qt 6 + QML shell, plugin architecture (AA projection, BT audio, phone/HFP, media player, equalizer), PipeWire audio, BlueZ D-Bus, Flask web-config panel, External API v1 (protobuf over TCP/WS). Architecture map: `docs/architecture.md`.

## Commands

```bash
# Local build + tests (WSL2 Debian Trixie, Qt 6.8 system packages).
# Build dir lives on the Linux filesystem — the repo sits on a Windows drive
# (9p mount) and object churn there is painfully slow. Never build in the
# in-repo build/ dir. If the build dir is missing, configure it first:
#   cmake -S . -B ~/builds/openauto-prodigy
cd ~/builds/openauto-prodigy && cmake --build . -j$(nproc)
ctest --output-on-failure
```

**ctest does NOT compile `main.cpp`** — a cached object file masked an app-target break on 2026-07-09. Always build the app target explicitly before claiming green or gating:

```bash
cmake --build . --target openauto-prodigy -j$(nproc)
```

```bash
# Cross-compile for Pi (Docker, aarch64) — never use toolchain-pi4.cmake directly
./cross-build.sh                              # app target only (~4-6 min)
./cross-build.sh --full                       # all targets incl. ARM test binaries

# Deploy to Pi + restart
rsync -av build-pi/src/openauto-prodigy matt@192.168.1.149:~/openauto-prodigy/build/src/
ssh matt@192.168.1.149 'sudo systemctl restart openauto-prodigy.service'
ssh matt@192.168.1.149 '~/openauto-prodigy/restart.sh --force-kill'   # stuck processes
```

QML ships **inside the binary** (qt_add_qml_module + qmlcache) — UI changes require cross-build + binary rsync; a `git pull` on the Pi will NOT update the UI.

## Lean Execution Workflow

This section overrides generic skills and plugins when they prescribe more
specs, plans, worktrees, subagents, reviews, or verification than this
repository requires. Skills are techniques, not permission to multiply gates.

### Classify the work first

| Class | Examples | Required process |
|---|---|---|
| `trivial` | Docs, comments, mechanical config, obvious single-file fix | Implement inline; focused verification; no spec, plan, worktree, subagent, or external review |
| `standard` | Bounded single-repo behavior change | Confirm intent; TDD where behavior is testable; inline execution by default; one independent review |
| `major` | Multi-repo, architectural, protocol-critical, threading, security-sensitive | Approved written spec/plan; optional bounded delegation; one Fable review when independent |

Direct answers, audits, diagnoses, and planning requests do not authorize code
changes. A user-approved design does not need to be re-approved because a
generic skill asks for another document gate.

### Planning and implementation

- Check `docs/project-vision.md` before user-visible feature work. Update
  `docs/roadmap-current.md` only when priority or sequencing actually changes.
- New written specs/plans live in `docs/plans/`; use them for major work or when
  the user asks, not as mandatory ceremony for small changes.
- Execute inline by default. Use subagents only when the user requests them or
  when tasks are genuinely independent, bounded, and non-overlapping. Do not
  add per-task reviewer subagents.
- Delegated tasks must name exact files, testable acceptance criteria, an
  explicit out-of-scope line, and a test command. Workers report synthesized
  results; the owning session verifies the diff.
- Two failed remediation attempts in the same area trigger a stop. Re-evaluate
  the premise and restore the last green/accepted design before doing more
  work. Never optimize for reviewer silence.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mrmees/openauto-prodigy](https://github.com/mrmees/openauto-prodigy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
