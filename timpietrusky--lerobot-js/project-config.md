---
trigger: always_on
description: **lerobot.js** is a TypeScript/JavaScript implementation of Hugging Face's [lerobot](https://github.com/huggingface/lerobot) robotics library. Our goal is to bring state-of-the-art AI for real-world robotics directly to the JavaScript ecosystem, enabling robot control without any Python dependencies.
---

# LeRobot.js Conventions

## Project Overview

**lerobot.js** is a TypeScript/JavaScript implementation of Hugging Face's [lerobot](https://github.com/huggingface/lerobot) robotics library. Our goal is to bring state-of-the-art AI for real-world robotics directly to the JavaScript ecosystem, enabling robot control without any Python dependencies.

### Vision Statement

> Lower the barrier to entry for robotics by making cutting-edge robotic AI accessible through JavaScript, the world's most widely used programming language.

## Core Rules

- **Never Start/Stop Dev Server**: The development server is already managed by the user - never run commands to start, stop, or restart the server
- **Package Manager**: Always use `pnpm` for package management - never use `npm` or `yarn` in documentation, scripts, or commands
- **Python lerobot Faithfulness**: Maintain exact UX/API compatibility with Python lerobot - commands, terminology, and workflows must match identically
- **Serial API Separation**: Always use `serialport` package for Node.js and Web Serial API for browsers - never mix or bridge these incompatible APIs
- **Minimal Console Output**: Only show essential information - reduce cognitive load for users
- **Hardware-First Testing**: Always validate with real hardware, not just simulation
- **Library/Demo Separation**: Standard library handles hardware communication, demos handle storage/UI concerns - never mix these responsibilities
- **No Code Duplication**: Use shared utils, never reimplement the same functionality across files
- **Direct Library Usage**: End users call library functions directly (e.g., `calibrate()`, `teleoperate()`) - avoid unnecessary abstraction layers
- **Comments**: Write about the functionality, not what you did. We only need to know what the code is doing to make it more easy to understand, not a history of the changes
- **No Reference Comments**: Never write comments like "same pattern as calibrate.ts", "matches Node.js", "copied from X", etc. Comments should explain what the code does, not where it came from or what it's similar to
- **ABSOLUTELY FORBIDDEN: No Change Explanation Comments**: NEVER EVER add comments explaining what you changed, what's new, what's updated, or why you removed something. This is a standard library - there is no "new API", no "old way", no "updated approach". Just code that does what it does. Examples of STRICTLY FORBIDDEN comments:

  - `// Create teleoperation process using new API`
  - `// Updated API to match Node.js`
  - `// New extensible architecture`
  - `// Breaking change from old API`
  - `// React import not needed with modern JSX transform`
  - `// Removed unused import`
  - `// Note: Connection manager registration removed for build compatibility`
  - `// Card components not needed in this file`
  - `// SerialPortRequestOptions unused in current implementation`

  **VIOLATION OF THIS RULE IS NOT TOLERATED.** Just make the change cleanly without explaining it in comments. If you catch yourself writing "new", "old", "updated", "changed", "removed", "added" in a comment - DELETE IT.

## Project Goals

### Primary Objectives

1. **Native JavaScript/TypeScript Implementation**: Complete robotics stack running purely in JS/TS
2. **Feature Parity**: Implement core functionality from the original Python lerobot
3. **Web-First Design**: Enable robotics applications to run in browsers, Edge devices, and Node.js
4. **Real-World Robot Control**: Direct hardware interface without Python bridge
5. **Hugging Face Integration**: Seamless model and dataset loading from HF Hub

### Core Features to Implement

- **Pretrained Models**: Load and run robotics policies (ACT, Diffusion, TDMPC, VQ-BeT)
- **Dataset Management**: LeRobotDataset format with HF Hub integration
- **Simulation Environments**: Browser-based robotics simulations
- **Real Robot Support**: Hardware interfaces for motors, cameras, sensors
- **Training Infrastructure**: Policy training and evaluation tools
- **Visualization Tools**: Dataset and robot state visualization

## Technical Foundation

### Core Stack

- **Runtime**: Node.js 18+ / Modern Browsers
- **Language**: TypeScript with strict type checking
- **Build Tool**: Vite (development and production builds)
- **Package Manager**: pnpm
- **Module System**: ES Modules
- **Target**: ES2020

## Architecture Principles

### 1. Platform-Appropriate Design Philosophy

**Each platform should leverage its strengths while maintaining core robotics compatibility**

#### Node.js: Python lerobot Faithfulness

- **Identical Commands**: `npx lerobot find-port` matches `python -m lerobot.find_port`
- **Same Terminology**: Use "MotorsBus", not "robot arms" - keep Python's exact wording
- **Matching Output**: Error messages, prompts, and flow identical to Python lerobot
- **Familiar Workflows**: Python lerobot users should feel immediately at home
- **CLI Compatibility**: Direct migration path from Python CLI


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TimPietrusky/lerobot.js](https://github.com/TimPietrusky/lerobot.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
