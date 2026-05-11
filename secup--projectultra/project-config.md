---
trigger: always_on
description: **If this is a new/fresh session, do this FIRST before any work:**
---

# ProjectUltra - HF Sound Modem

## FRESH SESSION? START HERE

**If this is a new/fresh session, do this FIRST before any work:**

1. **Read AI collaboration playbook:** `cat docs/AI_COLLABORATION.md` - **MANDATORY** - How to work with Codex (the other AI on this project), when to involve it, brief format, verification gates, autonomous-mode rules
2. **Read project goals:** `cat docs/PROJECT_GOALS.md` - Mission, priorities, and task filter
3. **Read current agent/project state:** `cat docs/AGENT_CURRENT_STATE.md` - Current automation and handoff context
4. **Check known bugs:** `cat docs/KNOWN_BUGS.md` - Active bugs you must not re-discover
5. **Check recent changes:** `git log --oneline -10` - See recent commits

**Before modifying ANY code, read:**
- `docs/AI_COLLABORATION.md` - Required workflow with Codex for non-trivial changes
- `docs/PROJECT_GOALS.md` - Mission, priorities, throughput/reliability targets, and agent task rule
- `docs/INVARIANTS.md` - Critical rules that MUST NOT be violated (causes subtle bugs if ignored)

**Autonomous agent work:**
- Use `docs/AGENTIC_DEVELOPMENT.md` and one task file in `agents/queue/`; do not work from an open-ended prompt.
- Use `docs/PROJECT_GOALS.md` to keep work aligned with the modem mission and current priorities.
- Use `docs/AGENT_TASK_BACKLOG.md` for approved task candidates and acceptance criteria.
- Use `docs/AGENT_DEDICATED_ENV_MACOS.md` for MacBook dedicated-agent setup.
- Use `docs/AGENT_CURRENT_STATE.md` to recover compacted/lost agent-system context.
- Run `./agents/run_local_gate.sh` before claiming a task is done.
- Use `./agents/run_hardware_smoke.sh` for PHY/ARQ/audio-path changes and respect the hardware lock.
- Do not grant agents unrestricted shell access; use repo-scoped allowlists from `agents/permissions/`.

**This project has durable documentation files.** They exist because context was lost repeatedly, causing rework. USE THEM.

---

## Hardware Audio Calibration - Mac <-> Pi 5 Test Rig

**Current known-good calibration (2026-04-29):**
- Mac USB soundcard: `Sound Blaster Play! 3`
- Pi USB soundcard: ALSA card 0, `USB Audio Device`
- Mac volume: output `71`, input `60`
- Pi mixer: `Speaker` 65% (`-13.00 dB`), `Mic Capture` 57% (`+8.00 dB`), `Auto Gain Control` off
- Synthetic channel hardware tests: use `--inject --inject-gain 0.70`

**Apply calibration before hardware tests:**
```bash
osascript -e 'set volume input volume 60' -e 'set volume output volume 70'
ssh -i "$HOME/.ssh/id_pi5" math@pi5tester \
  "amixer -D default sset 'Auto Gain Control' off && \
   amixer -D default sset 'Speaker' 65% && \
   amixer -D default sset 'Mic' capture 57%"
```

**Verify raw audio before modem tests:**
```bash
SSH_KEY="$HOME/.ssh/id_pi5" ./tools/check_hw_audio_path.sh
```

Expected calibrated raw levels:
- Pi -> Mac: RMS about `0.124`, peak about `0.303`, per-channel rough frequency near `1 kHz`
- Mac -> Pi: RMS about `0.249`, peak about `0.408`, per-channel rough frequency near `1 kHz`
- Acceptable target: RMS `0.05-0.25`, peak `0.15-0.80`
- Too hot: peak above `0.90`; this risks ADC/DAC clipping and invalid fading-test results
- Too low/silent: RMS near `0.0003` or below; check cable/device selection

Last verified captures:
- `/tmp/ultra_audio_path_20260429_220218/pi_to_mac_capture.wav`
- `/tmp/ultra_audio_path_20260429_220218/mac_to_pi_capture.wav`

Last post-calibration modem sweep:
- Good injected, 1 KB, R1/2, SNR 20/15/12: pass, `0` retx, `/tmp/ultra_hw_20260429_220250`, `/tmp/ultra_hw_20260429_220341`, `/tmp/ultra_hw_20260429_220445`
- Moderate injected, 1 KB, R1/2, SNR 20/15/12: pass with `4/7/8` retx, `/tmp/ultra_hw_20260429_220538`, `/tmp/ultra_hw_20260429_220717`, `/tmp/ultra_hw_20260429_220828`
- Moderate injected, 1 KB, R1/4, SNR 15/12: pass with `8/8` retx, `/tmp/ultra_hw_20260429_221117`, `/tmp/ultra_hw_20260429_221240`
- Good injected, 5 KB, R1/2, SNR 15: pass with `13` timeout retx, `/tmp/ultra_hw_20260429_221423`

Post ACK/control robustness patch checks:
- AWGN injected, 1 KB, R1/2, SNR 15: pass, `0` retx, `/tmp/ultra_hw_20260429_222350`
- Good injected, 1 KB, R1/2, SNR 15: pass, `0` retx, `/tmp/ultra_hw_20260429_222920`
- Moderate injected, 1 KB, R1/2, SNR 15: pass, `0` retx, `/tmp/ultra_hw_20260429_223017`
- Moderate injected, 1 KB, R1/4, SNR 15: pass, `0` retx, `/tmp/ultra_hw_20260429_223253`
- Moderate injected, 1 KB, R1/2, SNR 12: pass with `7` retx, `/tmp/ultra_hw_20260429_223113`
- Moderate injected, 1 KB, R1/4, SNR 12: pass with `5` retx, `/tmp/ultra_hw_20260429_223754`
- Good injected, 5 KB, R1/2, SNR 15: pass with `4` retx, `/tmp/ultra_hw_20260429_223926`

Corrected two-sided Pi/Mac rebuild checks:
- Good injected, 1 KB, R1/2, SNR 15: pass, `0` retx, `/tmp/ultra_hw_20260429_224520`
- Moderate injected, 1 KB, R1/2, SNR 15: pass, `0` retx, `/tmp/ultra_hw_20260429_224612`
- Good injected, 5 KB, R1/2, SNR 15: pass with `4` timeout retx, `/tmp/ultra_hw_20260429_224658`; BRAVO failed the original seq32-35 data burst (`CW[0..3]: FAIL`) and decoded the retransmissions, so this is data-side loss, not ACK/control loss.

Final ACK/control + burst/data-acquisition checks:
- AWGN injected, 1 KB, R1/2, SNR 15: pass, `0` retx, `/tmp/ultra_hw_20260429_230135`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [secup/ProjectUltra](https://github.com/secup/ProjectUltra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
