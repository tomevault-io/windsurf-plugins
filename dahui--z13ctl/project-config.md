---
trigger: always_on
description: `z13ctl` is a Linux CLI for controlling RGB lighting, fan curves, TDP (PPT power
---

# z13ctl — Project Context for Claude

## What this project is

`z13ctl` is a Linux CLI for controlling RGB lighting, fan curves, TDP (PPT power
limits), and system settings on the 2025 ASUS ROG Flow Z13 via Linux hidraw,
asus-wmi sysfs, and asus-armoury firmware-attributes interfaces.
It uses the ASUS Aura HID protocol reverse-engineered from g-helper.
Module path: `github.com/dahui/z13ctl`. Binary name: `z13ctl`. License: Apache 2.0.

## Package layout

```
api/                         Public client API submodule (github.com/dahui/z13ctl/api)
  go.mod                     Separate module; stdlib only; importable by z13gui and external tools
  types.go                   State, LightingState, FanCurvePoint, FanCurveState, TDPState, UndervoltState (exported protocol types)
  client.go                  SocketPath, Send*, Subscribe (all client functions)
  example_test.go            testable examples for all Send* and Subscribe functions
main.go                      entry point
cmd/                         Cobra subcommands
  root.go                    root command, Version var, dryRunFlag, deviceFlag, noButtonFlag
  apply.go                   apply lighting effect
  brightness.go              set brightness only
  daemon.go                  start the daemon (z13ctl daemon)
  list.go                    list hidraw devices
  off.go                     turn lighting off
  profile.go                 get/set performance profile (platform_profile sysfs); "custom" virtual profile
  batterylimit.go            get/set battery charge limit (power_supply sysfs)
  bootsound.go               get/set POST boot sound (asus-armoury firmware-attributes)
  paneloverdrive.go          get/set panel refresh overdrive (asus-armoury firmware-attributes)
  fancurve.go                get/set/reset custom fan curves (hwmon sysfs)
  tdp.go                     get/set/reset TDP power limits (asus-nb-wmi PPT sysfs)
  undervolt.go               get/set/reset CPU Curve Optimizer offsets via ryzen_smu
  status.go                  display system status (temperature, fans, profile, TDP, battery)
  setup.go                   install udev rules; applySysfsPerms helper (HID, hwmon, PPT, firmware-attributes, ryzen_smu)
internal/
  aura/                      Aura HID protocol implementation
    aura.go                  Writer interface + Init/SetPower/SetBrightness/SetMode/Apply/TurnOff
    modes.go                 Mode and Speed constants + ModeFromString/SpeedFromString
  cli/                       CLI helpers shared by cmd/ and internal/daemon/
    cli.go                   package doc file only
    colors.go                named color table, ResolveColor, PrintColorList
    parse.go                 ParseColor, ParseBrightness
    sysfs.go                 FindProfilePath, SetProfile, FindBatteryThresholdPath, FindBootSoundPath, SetBootSound, FindPanelOverdrivePath, SetPanelOverdrive, FindAPUTemperaturePath, ReadAPUTemperature, FindBatteryCapacityPath
    fan.go                   hwmon discovery, fan curve read/write (both fans), RPM read, mode control, ParseFanCurve, SetAllFansFullSpeed
    tdp.go                   PPT sysfs helpers, safety constants (TDPMin/TDPMaxSafe/TDPMaxForced), ReadAllPPT, SetTDP, ResetTDP
    smu.go                   SMU sysfs communication: SMUAvailable, SMUProbeUndervolt, SendSMUCommand, response codes
    undervolt.go             Curve Optimizer commands: SetCurveOptimizer, ResetCurveOptimizer, ValidateCOValues, safety limits
    undervolt_test.go        encodeCOValue, ValidateCOValues, smuResponseError tests
    fan_test.go              ParseFanCurve, FanModeName tests
    dryrun.go                DryRunApply, DryRunOff, DryRunBrightness, DryRunProfile, DryRunBatteryLimit, DryRunBootSound, DryRunPanelOverdrive, DryRunFanCurve, DryRunFanCurveReset, DryRunTdp, DryRunTdpReset, DryRunUndervolt, DryRunUndervoltReset
  daemon/                    long-running daemon (socket server, state, button watcher)
    daemon.go                Package doc, Daemon struct, Run(), getListener() (socket activation)
    state.go                 XDG state file persistence (uses api.State/api.LightingState)
    button.go                evdev button watcher (KEY_PROG3 / Armoury Crate button on 2025 Z13)
    server.go                JSON request handler; handleConn(), dispatch(), command handlers
    resume.go                DBus logind PrepareForSleep watcher; turns off lightbar on sleep, restores lighting + volatile state on resume
    client.go                Redirect comment only — client functions live in api/
  hid/
    hid.go                   package doc file only
    device.go                Device type, Write, SetFeature, Paths, Descriptions, Close
    scan.go                  FindDevice, ListDevices, sysfs discovery, hasAuraReport
    export_test.go           test-only exports: NewTestDevice, NewTestDeviceAnon,
                             UeventToDevPath, DeviceNameFromUevent
contrib/
  systemd/user/
    z13ctl.socket            systemd user socket unit (socket activation, %t/z13ctl/z13ctl.sock)
    z13ctl.service           systemd user service unit (Type=notify, Restart=on-failure)
  systemd/system/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dahui/z13ctl](https://github.com/dahui/z13ctl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
