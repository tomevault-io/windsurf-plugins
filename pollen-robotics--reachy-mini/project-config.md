---
trigger: always_on
description: This guide helps AI agents assist users in developing Reachy Mini applications.
---

# Reachy Mini Development Guide for AI Agents

This guide helps AI agents assist users in developing Reachy Mini applications.

---

## Agent Behavior

### FIRST: Check for agents.local.md

**Before doing anything else**, search for `agents.local.md` in the current directory:

```
IF agents.local.md exists:
    Read it immediately
    It contains user configuration and session context
ELSE:
    → Run skills/setup-environment.md to set up the environment
```

This file stores the user's robot type, preferences, and setup status. Always check it first.

### Be a Teacher

Unless the user explicitly requests otherwise:
- Explain concepts as you go
- Encourage questions ("Let me know if you'd like more detail on any of this")
- Guide non-technical users through each step
- Don't assume prior knowledge

### Two App Flavours — Default to JS, Python for developers

Reachy Mini supports two app types. **Default to a JS (Live/Web) app** unless the user explicitly wants on-robot Python, or has a need that only Python can cover (heavy on-robot compute, rich hardware access, deterministic real-time control loops, or bundled offline LAN tooling).

| Flavour | Default for | Where it runs |
|---|---|---|
| **JS app (recommended)** | End-users, shareable-by-URL experiences, anyone who wants "open a link, use the robot". Remote launch, zero-install, streaming A/V UIs. | Static HF Space; reaches the robot over WebRTC via the central signaling server. |
| **Python app** | Developer tools, on-robot control loops, heavy motion sequencing, offline/LAN. | Robot owner's machine (laptop / CM4), optionally with a bundled web UI. |

Both coexist — a Python app can bundle a browser UI, and a JS app can call the Python daemon's REST API.

**When unsure, start JS.** If the user later discovers they need on-robot compute they can graduate to a Python app; the reverse migration is rarely needed.

Confirm the choice with the user up front, then jump to the corresponding section:
- JS path → [Live/Web/JS Apps](#livewebjs-apps)
- Python path → instructions below

---

**Python path (for developers):**

- **NEVER create app folders manually** — use `reachy-mini-app-assistant`.
- **If a command fails**, ask the user to run it in their terminal — don't attempt complex workarounds.

```bash
# Default template (minimal app - good for most cases):
reachy-mini-app-assistant create <app_name> <path> --publish

# Conversation template (for LLM integration, speech, making robot talk):
reachy-mini-app-assistant create --template conversation <app_name> <path> --publish
```

Python apps put web UIs in `static/`. See `skills/create-app.md` for details.

### Always Create plan.md Before Coding

Before implementing any app:
1. Create `plan.md` in the app directory
2. Write your understanding of what the user wants
3. List your technical approach
4. Ask clarifying questions and provide answer fields inside `plan.md`
5. Wait for answers before coding

### Keep Notes in agents.local.md

Use `agents.local.md` to store:
- User's robot type (Lite/Wireless)
- Environment preferences
- Useful context for future sessions
- Keep it concise

---

## Robot Basics

**Reachy Mini** is a small expressive robot:

| Component | Description |
|-----------|-------------|
| **Head** | 6 DOF: x, y, z, roll, pitch, yaw (via Stewart platform) |
| **Body** | Rotation around vertical axis |
| **Antennas** | 2 motors, also usable as physical buttons |

**Hardware variants:**
- **Lite**: USB connection to laptop (full compute power)
- **Wireless**: Onboard CM4, connects via WiFi (limited compute)

---

## SDK Essentials

### Connection

```python
from reachy_mini import ReachyMini

with ReachyMini() as mini:
    # Your code here
```

### Two Motion Methods

| Method | Use when |
|--------|----------|
| `goto_target()` | **Default** - smooth interpolation for gestures that last at least 0.5s each |
| `set_target()` | Real-time control loops (e.g. tracking) at 10Hz+ |

### Basic Example

See and run `examples/minimal_demo.py` - demonstrates connection, head motion, and antenna control.

### Before Writing Code

- Read `docs/source/SDK/python-sdk.md` for API overview
- Skim `src/reachy_mini/reachy_mini.py` for method signatures and docstrings
- Check `examples/` for runnable code patterns

---

## Live/Web/JS Apps

> ## START HERE: clone `webrtc_example` and modify it.
>
> **Live Space (run it, read the source):** https://huggingface.co/spaces/cduss/webrtc_example
>
> This is the **canonical reference implementation** 
>
 **Mimicking what is done in this example is the fastest path to a working app.**

Don't necessarily clone it verbatim — feel free to trim panels, remove features, and tweak the UI as needed. But the core patterns (SDK wiring, event handling, session management, media flow, responsive mobile layout) are already solved there; start by understanding how it works before scaffolding something new. **Default to a mobile-friendly responsive design** (most users open Spaces on phones) — see the section below.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pollen-robotics/reachy_mini](https://github.com/pollen-robotics/reachy_mini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
