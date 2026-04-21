---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Educational template repository for Discord bots that communicate with Luxonis OAK-D cameras (DepthAI 3.x) on Raspberry Pi 5. Students learn to build smart object systems with computer vision and interactive communication.

**Target Hardware:** Three Raspberry Pi 5s (16GB each) named `orbit`, `gravity`, `horizon` — each with an OAK-D camera. SSH via `ssh orbit`, `ssh gravity`, `ssh horizon`.

**Core concept:** Cameras as conversational agents — reconfigure and query them in real-time through Discord, not just passive sensors configured once via SSH.

## Workflow Orchestration

### 1. Plan Mode Default

- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately - don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy

- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One task per subagent for focused execution

### 3. Self-Improvement Loop

- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project

### 4. Verification Before Done

- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)

- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes - don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing

- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests - then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

### Task Management

1. **Plan First**: Write plan to `tasks/todo.md` with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to `tasks/todo.md`
6. **Capture Lessons**: Update `tasks/lessons.md` after corrections

### Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimal Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

## Development Environment

```bash
# Activate shared venv (REQUIRED before running any script)
source /opt/oak-shared/venv/bin/activate
# Or use alias: activate-oak

# Key packages: depthai, depthai-nodes, opencv-python, numpy, discord.py, requests, aiohttp, python-dotenv
```

**Critical versions** (see `docs/WORKING_VERSIONS.md`): depthai 3.3.0, depthai-nodes 0.3.7, opencv-contrib-python 4.10.0.84, numpy 1.26.4 (numpy <2.0 required).

**Secrets:** Each user has `~/oak-projects/.env` with `DISCORD_WEBHOOK_URL` and `DISCORD_BOT_TOKEN`. Never commit `.env` files.

## Running Detectors

Each Pi has ONE camera — only ONE detector script should run at a time. Run with `--discord` so others see when the camera is free.

```bash
# Person detection (YOLO v6, COCO class 0)
python3 person_detector.py --discord --log --threshold 0.6
python3 person_detector.py --display              # requires X11/VNC

# Fatigue detection (YuNet + MediaPipe landmarks, EAR + head pose)
python3 fatigue_detector.py --dm                  # Discord DM notifications
python3 fatigue_detector.py --dm --dm-quiet       # only alert on fatigue

# Gaze direction (YuNet + head pose + gaze estimation ADAS)
python3 gaze_detector.py --display --log

# Whiteboard OCR - detection only (finds text regions)
python3 whiteboard_reader.py --discord --display

# Whiteboard OCR - full (detection + recognition, reads text content)
python3 whiteboard_reader_full.py --discord --log
```

Stop any detector with `Ctrl+C`. The systemd service (`sudo systemctl status person-detector`) can auto-start the person detector on boot.

## Code Architecture

### Repository Structure

```
├── person_detector.py           # YOLO person detection (template baseline)
├── person_detector_with_display.py  # Person detection with OpenCV visualization
├── fatigue_detector.py          # EAR + head tilt fatigue monitoring
├── gaze_detector.py             # 3-stage gaze direction estimation
├── whiteboard_reader.py         # OCR text detection only
├── whiteboard_reader_full.py    # OCR detection + recognition (reads text)
├── discord_bot.py               # Interactive bot (!status, !detect, !screenshot, !whiteboard, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kandizzy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
