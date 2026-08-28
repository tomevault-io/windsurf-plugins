---
trigger: always_on
description: Persistent working rules for AI coding agents in the Tokimi Rover repository.
---

# AGENTS.md

Persistent working rules for AI coding agents in the Tokimi Rover repository.

> Status: V0.1 source/documentation audit completed 2026-08-19. Current software facts are recorded under `docs/CURRENT_*.md`. No physical hardware was re-tested during that audit.

## 1. Required read order

Before changing code, hardware documents, public behavior, or release claims, read:

1. `AGENTS.md`
2. `PROJECT_CONTEXT.md`
3. `HARDWARE_AS_BUILT.md`
4. `KNOWN_ISSUES.md`
5. `ROADMAP.md`
6. `docs/CURRENT_IMPLEMENTATION.md`
7. `docs/CURRENT_PINMAP.md`
8. `docs/CURRENT_API.md`
9. `docs/SAFETY.md`

Use `docs/BUILD_AND_FLASH.md` for the canonical build procedure and `docs/RELEASE_CHECKLIST.md` before release work.

## 2. Source-of-truth hierarchy

Use this priority order:

1. Repository code plus a successful build from the current commit — software behavior, GPIO initialization, routes, timing, and dependencies.
2. Measured/photographed current as-built hardware — physical wiring, voltage, part models, connector orientation, and behavior.
3. `docs/CURRENT_*.md` — latest reconciled source audit.
4. `HARDWARE_AS_BUILT.md` — historical physical configuration with verification boundaries.
5. `PROJECT_CONTEXT.md` — product intent and design rationale.
6. `ROADMAP.md` — future work only.

Never silently change code to match a document or rewrite physical history to match code. Report the conflict, evidence, safety impact, and proposed resolution first.

## 3. Architecture invariants

- Main rover controller: ESP32-S3 N16R8; motors, OLED, WS2812 lighting, rover HTTP control, and motor safety state.
- Camera node: GOOUUU ESP32-S3-CAM V1.5 with OV3660; camera capture, camera AP, and camera HTTP service only.
- The controllers are separate. Current code has no GPIO/UART/I²C/SPI/network application transport between them.
- Do not move motor control onto the camera board.
- Do not add an inter-controller link or camera dependency to STOP without explicit approval and a failure-mode review.
- Camera failure must never delay or block motor stop.
- Do not change GPIO assignments without approval and a complete board/peripheral/native-USB conflict audit.
- Preserve public routes unless a deliberate compatibility or safety change is approved and documented.

Canonical GPIO assignments are in `docs/CURRENT_PINMAP.md`; do not duplicate an unaudited pin table elsewhere.

## 4. Current safety facts

These facts are `CODE-CONFIRMED` and must not be described more strongly:

- Motor outputs start stopped, with both PWM duties zero, direction pins LOW, and TB6612 STBY LOW.
- The configured physical PWM ceiling is **80%**, not 50–60%.
- Default requested speed is 30%, producing approximately 24% physical duty on an outside/full-speed channel.
- The command watchdog threshold is 750 ms; main-loop scheduling means it is not a hard maximum stop deadline.
- The embedded browser sends un-awaited 250 ms movement heartbeats. An older in-flight movement request can arrive after STOP and resume motion.
- There is no soft start or enforced direction-change dead time.
- AP station loss stops a moving rover only when associated-station count becomes zero; control-page closure can leave the station associated.
- Missing/invalid movement and speed inputs stop the rover. Missing/invalid lighting or expression inputs do not.
- There is no motor-current, driver-temperature, battery-voltage, fault, stall, low-voltage, or thermal protection input.
- The TB6612 installation has a historical failure consistent with overload/thermal shutdown under sustained four-motor load.
- The reported motors are rated 3–7.2 V while the documented 2S supply can reach 8.4 V.

`docs/SAFETY.md` is mandatory operating context. Never call the browser STOP a certified emergency stop, claim a guaranteed 750 ms stop, or describe the drive subsystem as production-ready.

## 5. Safety-preserving implementation rules

- Keep STBY LOW and both PWM channels zero until every required motor output is initialized.
- Preserve full stop output on every existing safety path.
- Do not raise the PWM ceiling. A lower ceiling still requires measured validation.
- Any reversal fix must first stop both sides, enforce a named timed dead interval, then ramp deliberately.
- Safety-critical motor timing must not depend on synchronous HTTP, display, lighting, or camera work.
- Resolve STOP command ordering with explicit sequencing/latching; do not rely solely on browser event order.
- Keep timeouts and caps as named constants and log stop reasons.
- Lift wheels and keep a physical motor-power disconnect available during testing.
- Never connect 2S voltage directly to WS2812, a 5 V fan, or a controller 5 V input.
- Measure LM2596 output before connecting 5 V loads.

## 6. Configuration and secrets

- Rover local network configuration belongs in ignored `firmware/rover-controller/include/local_config.h`, created from `local_config.example.h`.
- Camera local network configuration belongs in ignored `firmware/camera-node/include/camera_config.h`, created from `camera_config.h.example`.
- Never commit deployment SSIDs, passwords, private endpoints, personal tokens, or generated configuration.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TokimiSpace/tokimi-rover](https://github.com/TokimiSpace/tokimi-rover) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
