---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Seeker Robot — a ROS 2 Jazzy robotics project with ESP32 microcontrollers communicating via micro-ROS. The system runs inside Docker and spans two workspaces: a ROS 2 workspace for high-level autonomy and a PlatformIO workspace for MCU firmware.

## Architecture

- **`ros2_ws/`** — ROS 2 colcon workspace. Packages live in `ros2_ws/src/`:
  - `mcu_msgs` — Custom ROS 2 message/service/action definitions shared between ROS 2 nodes and micro-ROS MCUs. Includes `HexapodCmd.msg`, `OledFrame.msg`, `DetectedObject.msg`, `DetectedObjectArray.msg`, `SeekObject.action`, and `PerformMove.srv`. Also mounted into `mcu_ws/platformio/extra_packages/` by Docker so micro-ROS firmware can use the same interfaces.
  - `seeker_description` — URDF/Xacro hexapod model and `robot_state_publisher` launch.
  - `seeker_gazebo` — Gazebo Harmonic simulation, sensor bridges, and simulation launch files (including `sim_ball_search.launch.py`, `sim_object_seek.launch.py`, `sim_integrated_medium.launch.py`). Use `gui:=false` for headless performance on integrated graphics.
  - `seeker_navigation` — Nav2, SLAM Toolbox, EKF configs. Mission planner split across `ball_searcher` and `object_seeker.py` — a YOLO-driven WANDER/SEEK/PERFORM_MOVE state machine exposed as the `SeekObject` Action Server. `find.py` is a CLI client for manual goals.
  - `seeker_voice` — The "Brain". `command_node` acts as a `SeekObject` Action Client, using Gemini (with heuristic fallback when offline) to map free-form commands to robot actions and COCO classes. `transcription_node` handles speech → text. Launch via `local_mic.launch.py` (host mic) or `esp32_mic.launch.py` (ESP32 mic stream).
  - `seeker_sim` — `fake_mcu_node`: simulates ESP32 gait + dance for testing without hardware.
  - `seeker_test_cmd_vel` — `velocity_node`: minimal cmd_vel driver for manual/auto drive testing. Launch via `manual_drive.launch.py` / `auto_drive.launch.py`.
  - `seeker_display` — OLED display nodes: `oled_sine_node` (animated sine wave demo) and `lcd_http_server` (shared helper that serves SSD1306 framebuffers over HTTP on port 8390 for the ESP32 `OledSubsystem`).
  - `seeker_media` — MP4 media player node (`mp4_player_node`): decodes video to 128×64 SSD1306 framebuffers streamed over HTTP and audio to 16 kHz PCM streamed to the ESP32 speaker, with A/V sync.
  - `seeker_tts` — Fish Audio TTS node plus a local-WAV playback topic, both re-served as an HTTP PCM stream for the ESP32 `SpeakerSubsystem`.
  - `seeker_vision` — YOLO object detection (`vision_node`, `gazebo_vision_node`) with an HSV fallback (`use_yolo:=false` for light-weight color detection), DeepFace emotion detection (`emotion_node`), and an MJPEG camera proxy (`cam_proxy`) that bridges the ESP32 camera stream to localhost. Three launch files: `mcu_cam.launch.py` (ESP32 camera via proxy), `gazebo_cam.launch.py` (Gazebo `/camera/image`), `local_cam.launch.py` (host webcam).
  - `seeker_web` — Browser dashboard with topic allowlist and rate params.
  - `test_package` — Minimal C++ ROS 2 node for workflow verification.

### Brain-Body action pattern

Autonomous search is implemented as a **ROS 2 Action** rather than a one-shot topic:

1. **Brain** (`command_node.py` in `seeker_voice`) receives a command (e.g. "hey hatsune find the ball over").
2. **Brain** sends a `SeekObject` Goal to the **Body** (`ball_searcher` in `seeker_navigation`).
3. **Body** runs the WANDER → SEEK → PERFORM_MOVE state machine and publishes feedback.
4. **Body** returns a Success/Fail result once the object is reached.
5. **Brain** announces the result via TTS.

- **`mcu_ws/`** — PlatformIO workspace for ESP32 firmware. Uses micro-ROS WiFi transport (Jazzy distro). Multi-project layout:
  - `platformio/platformio.ini` — Shared base config (board environments, build flags, library deps). All sketches inherit from this via `extra_configs`.
  - `platformio/network_config.ini` — Local network settings (WiFi creds, agent IP, static IP). Gitignored; copy from `network_config.example.ini`.
  - `src/<sketch>/` — Each sketch is a standalone PlatformIO project with its own `platformio.ini` and `src/` directory.
  - `lib/` — Shared libraries available to all sketches via `lib_extra_dirs`.
  - `libs_external/esp32/micro_ros_platformio/` — micro-ROS PlatformIO library (pre-vendored).
  - `platformio/extra_packages/` — Extra ROS packages (including `mcu_msgs`) needed at micro-ROS build time.
- **`docker/`** — Containerized dev environment:
  - `Dockerfile` — Multi-stage build (`base` → `dev`/`prod`). Base installs micro-ROS agent, PlatformIO, ROS 2 Jazzy, and vision dependencies (ultralytics, deepface, tensorflow+CUDA). `dev` adds Gazebo Harmonic, RViz, rqt.
  - `Dockerfile.init-bootstrap` — One-shot init container that chowns named volumes and seeds `libs_external` into the `mcu_lib_external` volume.
  - `docker-compose.yml` — Defines `ros2` (main dev container), `init-bootstrap` (one-shot volume init), and GPU-enabled profile services `ros2-nvidia` (NVIDIA) and `ros2-amd` (AMD). `COMPOSE_PROJECT_NAME` in `.env` isolates containers/volumes per worktree.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SeekerRobot/seeker-robot](https://github.com/SeekerRobot/seeker-robot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
