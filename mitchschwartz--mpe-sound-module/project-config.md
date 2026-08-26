---
trigger: always_on
description: *Last updated: 2026-08-22 (America/Toronto)*
---

# MPE-Module — agent orientation

*Last updated: 2026-08-22 (America/Toronto)*

**Product:** Raspberry Pi MPE sound module (Surge XT headless + patch browser UI).

**Before looper / Phase 2 work:** [`Documents/DIRECTION.md`](Documents/DIRECTION.md) · [`Documents/DECISIONS.md`](Documents/DECISIONS.md) · OM-Repo [`GROUNDING.md`](https://github.com/opsMachine/OM-Repo/blob/main/internal/projects/mpe-synth-launch/GROUNDING.md)

**Before adding any polling loop, watchdog tick, or timer:** CPU is the scarcest resource
on this appliance — a `python3` fork is ~400 ms on the Pi, so once every 5 s is **9% of a
core, forever**. Compute cost × cadence and put it in the PR. Rules and measured constants:
[`Documents/DECISIONS.md`](Documents/DECISIONS.md) § *2026-08-18 — CPU is the scarcest resource*.

---

## 🔊 Audio output safety — read before making sound

**Headphones or speakers may be connected, and Mitch may be wearing them.** You cannot tell from the appliance. A loud transient into headphones on someone's head causes permanent hearing damage, and can destroy a driver instantly. This is the one failure on this project that cannot be rolled back.

**The rule:**

1. **Use the quietest level that proves the thing.** Verifying that audio is flowing needs far less level than "sounds good." Default to barely audible and raise only if the test genuinely requires it.
2. **Above 50% output, stop and ask.** Explicitly check in with Mitch before any test that exceeds it, and say what you intend to run. Do not infer consent from an earlier "go ahead" — his headphones may be on now and were not before.
3. **Never raise a level to diagnose silence.** If you expect sound and hear none, the cause is almost always routing, a stopped service, or a wrong device — not gain. Turning it up to find out is exactly how the damage happens. Check `mpe-yolo jack-status`, `osc-check`, and the unit states first.
4. **Restore any level you change**, and say in your summary that you changed it.

**There are three gain stages in series.** A level that is safe at one is not safe end to end:

| Stage | Control |
|---|---|
| Surge patch output | OSC `/param/a/amp/volume`, `/param/b/amp/volume` (UDP 53280) |
| Looper | `MPE_SL_LOOP_GAIN`, `MPE_SL_LOOP_GAIN_LAW` |
| Hardware mixer | `MPE_DAC_VOLUME_DB` in `/etc/mpe/mpe.env` → `scripts/set-dac-volume.sh` (`amixer` on Sound Blaster **Speaker**) |

**Hardware output is the Sound Blaster Play! 3** (card index varies by hotplug — scripts detect by name). The playback control is **`Speaker`** — there is no `PCM` control on this card. Scale **0–88 raw**, dB ≈ `(raw − 88) × 0.5`. Appliance default: **`MPE_DAC_VOLUME_DB=-12`** (raw **64**). Previous defaults: 76 (−6 dB), 48 (−20 dB).

Treat the **dB figure as the real number**, not the percentage — `amixer`'s percentage is not perceived loudness. Read or set via:

```bash
./scripts/set-dac-volume.sh --show
./scripts/set-dac-volume.sh   # applies /etc/mpe/mpe.env
```

**Loops sum.** A per-loop gain that is fine alone is not fine with 16 playing at once. Bring level up *after* the loops are running, never before.

**You cannot infer the output device.** Cards 0 (Pi headphone jack), 2 and 3 (HDMI) also exist. Which one reaches Mitch's ears is not visible from the appliance — assume the worst case.

---

## Pi CLI (`mpe`)

**Use the global `mpe` CLI** (separate [`mpe-cli`](https://github.com/MitchSchwartz/mpe-cli) repo) for laptop → Pi operations. Do not run raw `ssh`, `scp`, or `rsync` when a subcommand exists — Cursor allowlists fixed `mpe` entrypoints instead of open SSH.

Install once: clone `mpe-cli`, run `./install.sh`, edit `~/.config/mpe/mpe.env` (`PI_HOST`, `PI_USER`, `SSH_KEY`). **Do not embed the CLI in this repo.**

| Command | Purpose |
|---------|---------|
| `mpe ping` | Connectivity check |
| `mpe status` | Service active/enabled summary |
| `mpe logs surge\|touch\|watchdog [-n N]` | Recent logs (max 200 lines) |
| `mpe osc-check` | Surge OSC ports + process |
| `mpe diagnose` | Full read-only Pi diagnostics |
| `mpe sysinfo` | Board, kernel/preempt, EEPROM, CPU governor, Surge RT limits, buffer latency |
| `mpe record [file] [fps]` | Touch UI screen capture |
| `mpe pull-videos [-o DIR] [--delete-source]` | Download demo videos |
| `mpe restart surge\|touch\|all` | Restart fixed systemd units |
| `mpe looper sl-clips [local\|pi]` | SooperLooper eval: generate 16 fixture WAVs (default: pi) |
| `mpe looper sl-smoke [local\|pi]` | SooperLooper eval: 16-loop load/trigger smoke (default: pi) |
| `mpe looper sl-restart [local\|pi]` | Restart SooperLooper on JACK + wire record path (default: pi) |

**Agent-safe (read-only):** `ping`, `status`, `logs`, `osc-check`, `diagnose`, `sysinfo`, `pull-videos` (skip `--delete-source` for zero writes), `looper sl-clips local` (fixture generation only).

**Writes / restarts:** `restart *`, `record`, `pull-videos --delete-source`, `looper sl-smoke`, `looper sl-restart` (restarts SooperLooper on Pi).

**Do not allowlist for agents:** `scp`/`rsync`, `deploy-all.sh`, `set-audio-profile.sh`, `set-surge-audio.sh`, `set-midi-sync.sh`, poweroff/reboot.

**Raw `ssh` — allowlisted since 2026-08-14, and that is not the same as encouraged.**
This line previously read "do not allowlist raw `ssh`." It was amended because

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MitchSchwartz/MPE-Sound-Module](https://github.com/MitchSchwartz/MPE-Sound-Module) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
