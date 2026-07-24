---
trigger: always_on
description: Detailed how-to guides for common tasks are maintained as skill files under `.github/skills/`. **Before performing any of the tasks below, you MUST open and read the corresponding skill file.** Do not rely on prior knowledge or assumptions — the skill file is the source of truth.
---

# Copilot Instructions for librealsense

## Skills

Detailed how-to guides for common tasks are maintained as skill files under `.github/skills/`. **Before performing any of the tasks below, you MUST open and read the corresponding skill file.** Do not rely on prior knowledge or assumptions — the skill file is the source of truth.

| Skill file | Read before |
|---|---|
| `.github/skills/add_api.md` | Adding API to librealsense SDK |
| `.github/skills/file_creation.md` | Adding files to librealsense SDK |
| `.github/skills/cpp_coding.md` | Adding or modifying CPP code |
| `.github/skills/build.md` | Building the project (CMake configure, compile, flags) |
| `.github/skills/testing.md` | Running, filtering, and debugging unit tests |
| `.github/skills/pytest-infra.md` | Migrating tests to pytest, modifying pytest/hub infrastructure, verifying Jenkins CI results |
| `.github/skills/pr-create.md` | Opening a pull request, pushing commits to it, updating its description |
| `.github/skills/pr-review.md` | Replying to review comments on a pull request |

If a skill file exists for the task at hand, follow its instructions precisely. New skills may be added to this folder over time — check its contents before assuming none applies.

## Git Workflow (quick reference)

These rules apply to all git operations.

- **Base branch**: `development`
- **Branch naming**: short descriptive name, **no username prefix** (e.g. `fix-platform-camera`, not `nir/fix-platform-camera`)
- **PR target**: `development` branch
- **Push to**: `fork` remote (if no `fork` remote, ask the user)
- **Commits**: short one-sentence message, no Co-Authored-By, plain `git commit -m "message"`

## Project Overview

**librealsense** is the Intel® RealSense™ cross-platform open-source SDK for working with Intel RealSense depth cameras (D400, D500 series and others). It provides C, C++, Python, C#, and other language bindings.

## Architecture

- **Core library** (`src/`): The `realsense2` shared/static library written in C++14 (public API requires only C++11)
- **Public API headers** (`include/librealsense2/`): C and C++ headers; version is defined in `rs.h`
- **Common UI code** (`common/`): Shared code for the viewer and graphical tools
- **Examples** (`examples/`): Sample applications demonstrating SDK usage
- **Tools** (`tools/`): Utilities like `realsense-viewer`, `fw-updater`, `enumerate-devices`, etc.
- **Wrappers** (`wrappers/`): Language bindings — Python (pybind11), C#, Unity, OpenCV, PCL, etc.
- **Unit tests** (`unit-tests/`): Proprietary Python-based test framework orchestrated by `run-unit-tests.py`
- **Third-party** (`third-party/`): Vendored dependencies (`rsutils`, `realsense-file`, `json`, `glfw`, etc.)
- **CMake modules** (`CMake/`): Build configuration, platform detection, and external dependency management

### Layer Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                        │
│  (User Applications, Tools, Examples, Language Wrappers)    │
├─────────────────────────────────────────────────────────────┤
│                    Public C/C++ API                         │
│         (include/librealsense2/, rs.h, rs.hpp)              │
├─────────────────────────────────────────────────────────────┤
│                     Core Library                            │
│    (Context, Device, Sensor, Stream, Frame Management)      │
├─────────────────────────────────────────────────────────────┤
│                 Processing Pipeline                         │
│  (Format Conversion, Post-Processing, Synchronization)      │
├─────────────────────────────────────────────────────────────┤
│                 Platform Abstraction                        │
│       (UVC, HID, USB abstractions, Backend Interface)       │
├─────────────────────────────────────────────────────────────┤
│              Platform-Specific Backends                     │
│    (Windows: WMF/WinUSB, Linux: V4L2/libusb, macOS)        │
└─────────────────────────────────────────────────────────────┘
```

## Code Conventions

- **Copyright header**: every new source file must include `# License: Apache 2.0. See LICENSE file in root directory.` followed by `# Copyright(c) <current year> RealSense, Inc. All Rights Reserved.` — always use the actual current year, not the year of a nearby file being used as a template
- The core library compiles as **C++14** (`cxx_std_14` — see `CMake/lrs_macros.cmake`)
- The public interface only requires **C++11** (`cxx_std_11`)
- Examples and wrappers generally use **C++11**
- CMake minimum version: **3.10** (3.16.3 when `BUILD_WITH_DDS` is enabled)
- Use the existing code style in surrounding files; the project does not enforce a formatter
- Logging uses EasyLogging++ (controlled by `BUILD_EASYLOGGINGPP` option)

## Code Change Discipline

These rules apply to every code change.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [realsenseai/librealsense](https://github.com/realsenseai/librealsense) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
