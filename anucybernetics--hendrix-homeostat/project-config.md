---
trigger: always_on
description: A Nerves-based cybernetic audio system inspired by W. Ross Ashby's homeostat,
---

# Hendrix homeostat project

A Nerves-based cybernetic audio system inspired by W. Ross Ashby's homeostat,
running on Raspberry Pi 5 to control a guitar feedback loop system.

## Project architecture

- **Target platform**: Raspberry Pi 5 (Nerves embedded system)
- **Audio interface**: Presonus Revelator io24 (USB audio/MIDI)
- **Control output**: MIDI commands to Boss RC-600 loop station
- **Audio flow**: guitar pickup → audio analysis → MIDI track control → effects
  loop

## Key implementation points

### Elixir/Nerves patterns

- use GenServers for audio monitoring and control logic
- leverage OTP supervision trees for fault tolerance
- use Nerves-specific libraries for audio/MIDI interface
- target is `:rpi5`, development target is `:host`, always specify the correct
  one e.g. `MIX_TARGET="host" mise exec -- mix test`
- **IMPORTANT**: Always use `mise exec -- mix ...` instead of just `mix ...` to
  ensure correct Elixir/Erlang versions

### Audio processing

- monitor RMS level (root mean square - primary metric for control)
- also calculate zero-crossing rate and peak (not currently used in control)
- implement threshold-based decision making (critical high >0.8, comfort zone
  0.2-0.5, critical low <0.05)
- balance between real-time responsiveness and system stability

### Control philosophy: Ashby-style ultrastability

The system implements a **double feedback loop** inspired by W. Ross Ashby's
homeostat:

**First-order loop (homeostasis):**

- RMS too high (≥0.8) → stop random track (damping)
- RMS too low (≤0.05) → start recording (excitation)
- Stable too long → clear track (anti-stasis)
- Controls only tracks 1-2 for simplicity

**Second-order loop (ultrastability):**

- When first-order control fails (e.g., oscillating >10 times between extremes
  in 100 samples over 60 seconds)
- System randomly changes track parameters:
  - Track 1: volume (5 levels) + speed (5 levels) - the "experimental" track
  - Track 2: volume (5 levels) only - the "anchor" track
- This creates 125 possible configurations to explore
- Like Ashby's uniselector, system searches for parameter combinations that
  achieve equilibrium
- Asymmetry aids stability (functional differentiation)

**Why it works:**

- Negative feedback maintains bounds
- Parameter adaptation enables learning through trial-and-error
- System discovers stability rather than being programmed for it
- Random search provides requisite variety to match disturbances

## Development workflow

- always set `MIX_TARGET=rpi5` for target builds, or use `:host` for local
  testing
- use `mise exec -- mix firmware` and `mise exec -- mix burn` for deployment to
  hardware
- test audio processing logic on `:host` target when possible before burning to
  device

## Testing

This project uses a backend abstraction pattern to enable testing on host
without hardware.

### Running tests on host

Most tests run on host without requiring hardware:

```bash
# Run all tests on host (default, works in standard shell)
MIX_TARGET=host mise exec -- mix test

# Run specific test file
MIX_TARGET=host mise exec -- mix test test/path/to/test.exs

# Run integration tests only
MIX_TARGET=host mise exec -- mix test test/integration/
```

### Running tests on target device

Target-only tests require real hardware and must be run on the Raspberry Pi. Use
the ht-mcp server for interactive SSH sessions:

```bash
# On target hardware with real devices
MIX_TARGET=rpi5 mise exec -- mix test --include target_only
```

To run commands (tests, IEx sessions, etc.) on the target device, use ht-mcp:

```elixir
# Create an SSH session to the target
mcp__ht-mcp__ht_create_session(command: ["ssh", "jimi.local"], enableWebServer: true)

# Execute commands on the target
mcp__ht-mcp__ht_execute_command(sessionId: "session-id", command: "mise exec -- mix test --only target_only")

# Or start an IEx session
mcp__ht-mcp__ht_send_keys(sessionId: "session-id", keys: ["mise exec -- iex -S mix", "Enter"])
```

### Test types

- **Pure functions**: AudioAnalysis tests run on both host and target
- **GenServer unit tests**: Use backend abstractions (InMemory MIDI, File audio)
- **Integration tests**: Deterministic tests that send messages directly to
  ControlLoop
- **Target-only tests**: Tagged with `@tag :target_only`, require real hardware

The test suite automatically excludes `:target_only` tests when running on host.
See `test/test_helper.exs` for configuration details.

## Task management

- use the backlog CLI tool via the project-manager-backlog agent for task
  tracking
- use the elixir-ash-phoenix-developer agent for Elixir-specific implementation
  work (note: this project doesn't use Ash or Phoenix, but the agent has strong
  Elixir expertise)

<!-- usage-rules-start -->
<!-- usage-rules-header -->

# Usage Rules

**IMPORTANT**: Consult these usage rules early and often when working with the
packages listed below. Before attempting to use any of these packages or to
discover if you should use them, review their usage rules to understand the
correct patterns, conventions, and best practices.

<!-- usage-rules-header-end -->

<!-- igniter-start -->

## igniter usage

_A code generation and project patching framework_

@deps/igniter/usage-rules.md

<!-- igniter-end -->

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ANUcybernetics/hendrix-homeostat](https://github.com/ANUcybernetics/hendrix-homeostat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
