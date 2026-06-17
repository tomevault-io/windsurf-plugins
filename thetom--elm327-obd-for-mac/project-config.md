---
trigger: always_on
description: Native macOS Ford Diagnostic Tool — a Rust CLI that communicates directly with ELM327 USB adapters for Ford-specific OBD-II and UDS diagnostics on Apple Silicon. No Wine, no Windows. See `PRD.md` for full product spec.
---

# CLAUDE.md

# Ford Diagnostic Tool — Autonomous Development Rules

Native macOS Ford Diagnostic Tool — a Rust CLI that communicates directly with ELM327 USB adapters for Ford-specific OBD-II and UDS diagnostics on Apple Silicon. No Wine, no Windows. See `PRD.md` for full product spec.

Core thesis:

Most Ford owners can't run FORScan on macOS without painful Wine hacks.
Build a native Rust tool that talks directly to ELM327 adapters and the truck's CAN bus.

This repository is developed using **test-driven, autonomous execution with strict guardrails**.

---

# Core Operating Principles

1. **Tests before implementation — mandatory.**
2. **Every piece of work must correspond to a GitHub Issue.**
3. Implement the **smallest working solution**.
4. **No unsafe unless justified** — if you write `unsafe`, comment why.
5. Keep architecture **simple and extensible**.
6. Keep the repository **compilable at all times**.
7. Maintain **organized documentation, repository maps, and test maps**.
8. Follow the **issue → failing tests → implementation → review → commit → close issue loop**.
9. Focus on **serial communication, ELM327 protocol, OBD-II, and Ford-specific diagnostics**.
10. If uncertain about a requirement, create a **GitHub Issue** and continue with other unblocked work.

---

# GitHub Issue-Driven Development (MANDATORY)

This project uses **GitHub Issues as the bug/task system**, modeled after Google Buganizer.

Every unit of work is treated as an issue.

This includes:

- features
- bugs
- implementation tasks
- investigations
- product questions
- refactors
- architectural decisions
- hardware compatibility findings

---

# Issue Requirement (MANDATORY)

Before starting any work:

1. Ensure a **GitHub Issue exists**
2. Assign the correct priority label
3. Assign the correct type label
4. Assign the correct area label(s)
5. Write failing tests
6. Only then may implementation begin

No task may be worked on without a corresponding issue.

---

# Priority Labels (Buganizer Style)

Use the following priorities:

- **P0** — blocking / critical / hardware-damaging potential / correctness failure
- **P1** — core product work (current phase)
- **P2** — normal development task
- **P3** — improvement / cleanup
- **P4** — polish / optional

Guidelines:

- P0 should be rare — reserve for things that could brick an ECU or corrupt CAN bus
- most work should be P1 or P2
- cleanup tasks are P3
- ideas or polish are P4

---

# Issue Labels

Issues should use labels for classification.

Priority labels:

- `P0`
- `P1`
- `P2`
- `P3`
- `P4`

Type labels:

- `type:feature`
- `type:bug`
- `type:task`
- `type:investigation`
- `type:question`
- `type:refactor`
- `type:hardware` — hardware compatibility finding or issue

Area labels:

- `area:serial` — serial port, TTYPort, macOS device handling
- `area:obd` — OBD-II protocol, PID decoding, DTC parsing, VIN
- `area:ford` — Ford module database, CAN address pairs, MS-CAN/HS-CAN
- `area:elm327` — ELM327 AT commands, protocol init, prompt handling
- `area:cli` — CLI binary, clap subcommands, user-facing output
- `area:simulator` — ELM327 simulator, PTY-based testing
- `area:bridge` — bridge forwarding, PTY ↔ serial
- `area:config` — YAML config, device detection settings
- `area:docs` — documentation, CLAUDE.md, PRD, DEVLOG

---

# Development Workflow (MANDATORY)

All work must follow this exact sequence.

1. Create or identify GitHub Issue
2. Confirm labels and priority are correct
3. Write **failing tests first**
4. Confirm tests fail for the expected reason
5. Implement the minimal code required
6. Run tests: `make test`
7. Run linter: `make lint`
8. Ensure compilation: `make build`
9. Run `codex-review`
10. Run `@roast review this change`
11. Fix any issues
12. Rerun tests and lint if code changed
13. Commit changes
14. Close the issue with resolution comments

No feature is complete without tests, compilation, linting, and review.

---

# Autonomous Execution Loop

When operating autonomously:

1. Review open GitHub Issues
2. Select the highest-priority unblocked issue
3. Write failing tests first
4. Implement the minimal solution
5. Run tests: `make test`
6. Run linter: `make lint`
7. Ensure compilation: `make build`
8. Run `codex-review`
9. Fix review findings
10. Run `@roast review this change`
11. Fix roast findings
12. Rerun tests, lint, and build if code changed
13. Commit changes
14. Close issue with resolution comments
15. Move to next issue

Do not skip the **tests-first** step.

---

# Definition of Done (MANDATORY)

A task or feature is complete **only when all conditions are met**.

1. A GitHub Issue exists and is the source of the work
2. Tests were written first
3. Tests pass (`make test`)
4. Code coverage thresholds pass
5. Code compiles successfully (`make build`)
6. Linting passes (`make lint` — clippy with `-D warnings`)
7. Codex review completed using `codex-review`
8. Roast agent signoff completed using `@roast`
9. Documentation updated if behavior, architecture, or tests changed
10. Issue closed with resolution comments

If any step is missing, the task is **not complete**.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheTom/elm327_obd_for_mac](https://github.com/TheTom/elm327_obd_for_mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
