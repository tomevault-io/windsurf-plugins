---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Prosthetic Arm Project — Claude Code Context

## Stack
- Firmware: ESP32 / PlatformIO / C++
- ML: Python 3.11, snnTorch, scikit-learn, numpy
- Power logging: Python + INA219 over I2C

## Conventions
- C++: follow Arduino style, use FreeRTOS tasks
- Python: type hints, docstrings on all functions
- Commit style: conventional commits (feat:, fix:, docs:)

## Key files
- firmware/include/config.h — all pin definitions, change here first
- ml/src/snn/ — the neuromorphic pipeline, our novel contribution
- firmware/src/main.cpp — FreeRTOS task structure

# Neuromorphic Prosthetic Control System

![CI](https://github.com/YOUR_USERNAME/prosthetic-arm/actions/workflows/ci.yml/badge.svg)
![Python](https://img.shields.io/badge/python-3.11-blue)
![Platform](https://img.shields.io/badge/platform-ESP32-red)
![Framework](https://img.shields.io/badge/framework-PlatformIO-orange)
![License](https://img.shields.io/badge/license-Academic-lightgrey)

> Real-time EMG-driven prosthetic hand control using event-driven Spiking Neural Networks — benchmarked against classical ML and deep learning baselines on latency, accuracy, and power draw.

**Final Year Engineering Project**

---

## Overview

This system captures surface EMG signals from the forearm, classifies intended hand gestures in real time using three competing ML pipelines, and drives an [InMoov 3D-printed right hand and forearm](https://inmoov.fr/hand-and-forarm/) via six servo motors. The core research contribution is a neuromorphic SNN classifier that operates on spike-encoded EMG features, targeting lower inference latency and power consumption than conventional approaches.

```
Forearm EMG  →  ESP32 ADC  →  Feature Extraction  →  Classifier  →  Servo Control
     ↑                                                                      ↓
  MyoWare 2.0                  RMS · MAV · ZCR · WL         InMoov right hand + forearm
  4 channels                   4th-order Butterworth         5× finger servos + 1× wrist
  1 kHz sampling               200 ms window / 50% overlap   FSR force feedback · INA219
```

---

## ML Pipeline Comparison

| Pipeline | Classifier | Target latency | Key advantage |
|---|---|---|---|
| Baseline | SVM · Random Forest | < 100 ms | Interpretable, fast to train |
| Deep | CNN · MLP (PyTorch) | < 80 ms | End-to-end feature learning |
| **Neuromorphic** | **Spiking Neural Net** | **< 50 ms** | **Event-driven, low power** |

All three pipelines are trained on the same labelled EMG dataset and evaluated identically — same test split, same inference hardware, same power measurement setup.

---

## Hardware

| Component | Part | Role | Interface |
|---|---|---|---|
| EMG sensors | MyoWare 2.0 ×4 | Forearm muscle signals | Analog → ADC1 |
| IMU | MPU-6050 | Wrist orientation | I2C `0x68` |
| Microcontroller | ESP32-WROOM-32 | Sampling · inference · control | — |
| Power monitor (logic) | INA219 | ESP32 + sensor rail power | I2C `0x40` |
| Power monitor (servo) | INA219 | Servo rail power | I2C `0x41` |
| Prosthetic hand | InMoov right hand (3D printed) | 5 independently actuated fingers | PWM |
| Prosthetic forearm | InMoov forearm (3D printed) | Houses 5× finger servos + 1× wrist servo | PWM |
| Finger servos | MG996R ×5 | Independent finger flexion via fishing line tendons | PWM |
| Wrist servo | MG996R ×1 | Wrist rotation (0–180°) | PWM |
| Force sensors | FSR 402 ×4 | Fingertip contact feedback | Analog → ADC1 |
| Co-processor *(opt.)* | Raspberry Pi 4 | Heavy ML · data logging | UART / WiFi |

### InMoov Hand & Forearm

The physical prosthetic is the open-source [InMoov right hand and forearm](https://inmoov.fr/hand-and-forarm/) designed by Gaël Langevin — a life-sized, 3D-printable robotic hand. Key mechanical details:

- **5 independently actuated fingers** — each driven by a dedicated servo motor housed in the forearm, connected to the finger via braided fishing line tendon (200 lb breaking strength) routed through transparent tubing
- **1 wrist servo** — controls wrist rotation, centred at 90° (flat position)
- **Tendon return springs** — provide tension for realistic finger extension
- **Servo bed** — printed enclosure inside the forearm that mounts all 6 servos
- **STL files** — [InMoov right hand](https://inmoov.fr/inmoov-stl-parts-viewer/?bodyparts=Right-Hand) · [Forearm assembly](https://inmoov.fr/hand-and-forarm/)
- **Print settings** — finger parts at 1.5 mm wall thickness; PLA or ABS; parts can be bonded with acetone, epoxy, or two-part plastic adhesive

---

## Repo Structure

```
prosthetic-arm/
│
├── firmware/                    # ESP32 C++ — PlatformIO project
│   ├── platformio.ini           # Board config, libs, 4 build environments
│   ├── include/
│   │   ├── config.h             # ← ALL pin/rate/address constants live here
│   │   ├── emg.h                # EMGReader — ADC ISR, filter, features
│   │   ├── imu.h                # IMUReader — MPU-6050, orientation
│   │   ├── power_monitor.h      # PowerMonitor — dual INA219
│   │   ├── fsr.h                # FSRReader — force sensing + safety
│   │   ├── servo_control.h      # ServoController — gesture → PWM

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ruth205/FinalProject](https://github.com/ruth205/FinalProject) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
