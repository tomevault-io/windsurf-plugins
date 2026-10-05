---
trigger: always_on
description: Eye tracking for the Steam Frame in VRChat via VRCFaceTracking (VRCFT). Read `README.md` for the user-facing docs and
---

# vrcft-steam-frame: notes for Claude Code sessions

Eye tracking for the Steam Frame in VRChat via VRCFaceTracking (VRCFT). Read `README.md` for the user-facing docs and
`CLAUDE.local.md` (git-ignored, if present) for machine-specific details (headset address, SSH key, paths).

## Current state (2026-09-27)

- Latest release **v0.2.2** (2026-09-28: review fixes, calibration voice fix, pinned frameeyeosc), prebuilt-DLL zip. Working end to end on the author's setup: per-eye gaze, blinks, both
  winks, wink assist; frameeyeosc auto-starts on the headset and survived a real reboot; `doctor.ps1` reports "All good".
- `setup.ps1 -Headset` (menu 2) was run for real on 2026-09-27 against an already-set-up Frame: SSH login, target kept, build no-op,
  unit refreshed, service active, doctor "All good"; a second run changed nothing. Still never exercised: the first-time path on a fresh
  headset (Rust install + first build). Treat that as a test when someone new tries it. The key authorisation by password was tested
  on 2026-09-28 with a throwaway key against the real headset: it used to pipe the public key into `ssh` (stdin), which made Windows
  OpenSSH unable to read the password (reported by a user); now the key goes in the command line, password auth is forced and a
  duplicate key is not appended. Gotcha when testing: a key file under `%TEMP%` can have an orphaned-SID ACL, and Windows ssh then
  ignores it ("bad permissions"); use `icacls <key> /inheritance:r /grant:r "$env:USERNAME:F"`. The `!` box in Claude Code is bash
  without a TTY: password prompts need a real PowerShell window.
- Open ideas: verify the gaze scale (frameeyeosc +-1 == +-45 deg -> radians) against a reference; a first-run check in `Start Here.cmd`
  that VRCFT is running (users forget to start it from Steam each session; the doctor catches it).

## Review fixes in 0.2.2 (2026-09-27/28, from an external review; all verified against the code first; released)

- `tune.py analyze` rejects a calibration whose per-eye open-closed range is < 0.15 or reversed (was a ZeroDivisionError or bad
  values), keeps the previous config, and says so. It now also recommends `wink.assist` + `assistMin` (other eye's both-closed p75 + 0.08).
- `tune.py record` checks first (status file) that VRCFT runs and frameeyeosc data is fresh, and speaks what is missing.
- `ModuleConfig.TryLoad` + `Validate`: a broken or invalid `steamframe-config.json` is not applied; the module keeps the last good
  config and reports `configError` (status file, log, doctor). Previously it silently fell back to defaults.
- Without fresh data from either source the module sets neutral eyes (open, centred) once, instead of replaying stale Steam Link values.
- Trace lines are written with `FormattableString.Invariant` (decimal-comma locales broke the CSV); `tune.py` skips malformed lines.
- `setup.ps1` sandbox mode (`-VrcftData`) no longer touches the real SteamVR settings (unless `-SteamVrSettings <file>`) or the firewall.
- Firewall: a *block* rule is handled separately (disable it + add allow, elevated); an allow rule alone does not beat a block rule.
- `headset-setup.sh` pins frameeyeosc to a reviewed commit (`REV`, override `FRAMEEYEOSC_REV`) and moves existing checkouts to it;
  `FRAMEEYEOSC_NO_BUILD=1` tests only the checkout logic (works locally with `HOME=<tmp>`).
- `Select-HeadsetTarget` only counts addresses of connected adapters (Windows lists a disconnected adapter's IP).
Follow-up review of f5ec145, also fixed in 0.2.2: `Validate` checks relationships (`maxFloor + minRange <= 1`, `minCeil > 0`) and the
adaptive clamp can no longer get a lower bound above 1 (fuzzed: 7269 valid configs x 200 samples through `LidCal.Map`, no throw, output 0..1);
a failed config read is retried every second (timestamp recorded only after success, each distinct error logged once; tested with a 3 s
exclusive lock); `tune.py analyze` maps with the configured `lid.deadband` and `wink.assistClosed` (`STEAMFRAME_CONFIG=<file>` overrides
the config path for tests). `settings_or_exit` validates those values (deadband 0..0.45, assistClosed 0..1, numbers only) with the
module's limits before any calculation, and `record`/`calibrate` run it before recording; broken files give a spoken error, nothing changes.
Calibration voice: `tune.py` used to start PowerShell + System.Speech for every prompt (~1.5 s idle, worse under VR load), so prompts
lagged their beeps and piled up; the user could not hear them (2026-09-28). `Speaker` keeps one engine for the whole recording (started in
the lead-in, 7-10 ms per prompt, a new prompt cancels a late one). Not caused by 0.2.2: the prompt code was unchanged since v0.2.1.
Not done yet: offline replay tests for the eyelid pipeline (the replays in this history were ad-hoc scripts).

## Working with the user

The user is in VR most of the time and cannot read text in the headset. Prefer spoken output (`Say` in `common.ps1`, `speak` in `tune.py`),
one-click entry points (`Start Here.cmd`), and short answers. They test in VRChat and report what the avatar does.

## Layout

- `module/` C# (net10.0) VRCFT module, version in `SteamFrameVRCFTModule.csproj` (`<Version>`). `SteamFrameVRCFTModule.cs` (logic, status

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hakumaguro/vrcft-steam-frame](https://github.com/hakumaguro/vrcft-steam-frame) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
