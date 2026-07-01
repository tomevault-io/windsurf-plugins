---
trigger: always_on
description: This repository is a macOS 27+ USB RNDIS tethering VM project. Future agents
---

# AGENTS.md

This repository is a macOS 27+ USB RNDIS tethering VM project. Future agents
should read this file before the README and treat the current
WireGuard-over-VZNAT architecture as the baseline.

## Project Shape

- The app is a SwiftUI Dock app, not a CLI `main.swift` entrypoint.
- The Xcode project is `RNDIS Tethering VM Passthrough.xcodeproj`.
- The main app target is `LinuxVirtualMachine` and builds a macOS app bundle.
- There is no host packet-tunnel extension target. The app does not create a
  host VPN, and does not inspect or forward packet payloads.
- Linux assets are not bundled. Users load the generated asset folder or select
  the kernel, initramfs, and raw disk image from the app UI.

## Architecture

- `TetheringStore` owns app state, VM lifecycle, USB accessory selection,
  WireGuard configuration state, and the console/event logs.
- `WireGuardConfigurationLoader` loads generated `wg-server.conf` and
  `wg-client.conf` files from the selected asset tree for preview/export.
  It must not hard-code WireGuard key material.
- `VMConfigurationFactory` builds the Linux VM configuration. The current
  baseline uses `VZLinuxBootLoader`, raw disk attachment, an XHCI USB
  controller, and `VZNATNetworkDeviceAttachment`.
- USB passthrough must stay on the public API path that passes an
  AccessoryAccess `AAUSBAccessory` into
  `VZUSBPassthroughDeviceConfiguration(device:)`.
- The guest owns packet forwarding by running a normal WireGuard peer on the
  Virtualization NAT private network and masquerading WireGuard client traffic
  out the USB RNDIS interface.
- The host side is intentionally manual: users configure the host WireGuard
  client from the generated client `.conf`. The official WireGuard macOS client
  currently has a connection bring-up issue in this validation flow; recommend
  `wireguard-go` for macOS validation. This app must not start, stop, or manage
  the host WireGuard tunnel.

## Data Path

The current baseline data path is:

```text
macOS WireGuard client
-> VZNAT guest endpoint UDP/51820
-> guest wg0
-> guest nftables masquerade
-> USB RNDIS upstream
```

- The current manual WireGuard test addresses are guest `10.100.0.1/24` and
  macOS host tunnel `10.100.0.2/24`; the guest peer should allow
  `10.100.0.2/32`.
  This is the WireGuard overlay address, not the guest `usb0` RNDIS DHCP
  address.
- The guest WireGuard server listens on UDP port `51820`.
- The app parses `RTPVM_WG_ENDPOINT=<guest-nat-ip>:51820` from serial console
  output. The BusyBox init network one-shot prints that marker after VZNAT
  `eth0` DHCP succeeds.
- `script/make_vm_assets` uses the host `wg` CLI from Homebrew's
  `wireguard-tools` to generate fresh WireGuard server/client keypairs, writes
  `script/assets/wireguard/wg-server.conf` and
  `script/assets/wireguard/wg-client.conf`, then builds the initramfs.
  The server config is copied into the RAM-backed guest as
  `/etc/wireguard/wg0.conf`.
- The generated initramfs runs BusyBox `init` as PID 1. Its `once` network
  action runs `wg-quick up wg0` after the VZNAT `eth0` DHCP step, then waits
  for fixed `usb0` RNDIS DHCP, source policy routing from `10.100.0.0/24` to
  the RNDIS default gateway, IPv4 forwarding, and scoped nftables masquerade
  from `wg0` to `usb0`.
- The generated client `.conf` acts as a WireGuard client and uses
  `<RTPVM_WG_ENDPOINT>` as a placeholder for the discovered guest VZNAT address.
  The client `.conf` uses IPv4 full-tunnel routing with
  `AllowedIPs = 10.100.0.0/24, 0.0.0.0/1, 128.0.0.0/1`. Keep the explicit
  overlay route so `10.100.0.1` remains reachable, and prefer split internet
  routes over `0.0.0.0/0` on macOS to avoid a bad `wg-quick` direct route over
  the VZNAT endpoint. IPv6 tunneling remains out of scope until RNDIS IPv6 is
  tested.
- The BusyBox init network one-shot brings up the VZNAT NIC `eth0` and runs
  `udhcpc` so the guest has an IPv4 endpoint address during early boot.
- The guest RNDIS interface is fixed to `usb0`. The app supports one
  passthrough RNDIS accessory per VM session; if that accessory detaches, the
  app restarts the VM instead of attaching a replacement into the same
  guest session.
- Guest NAT is based on `nftables` masquerade from `wg0` traffic to `usb0`.
- The setup NAT NIC provides the private host-to-guest network used for the
  WireGuard endpoint. Do not
  replace it with vmnet, bridged networking, route-command UI, or a host-side
  packet-tunnel provider.

## Directory Guide

- `LinuxVirtualMachine/App`: SwiftUI app entrypoint and top-level commands.
- `LinuxVirtualMachine/Views`: setup, USB, console, and WireGuard views.
- `LinuxVirtualMachine/Stores`: `TetheringStore` orchestration and
  `WireGuardConfigurationLoader` key/config loading and host config rendering.
- `LinuxVirtualMachine/Services`: AccessoryAccess monitor, VM configuration
  factory, and VM delegate glue.
- `LinuxVirtualMachine/Support`: file picker, clipboard, and runtime entitlement
  reader helpers.
- `LinuxVirtualMachine/GuestScripts`: currently empty. The generated RTPVM
  initramfs includes a server WireGuard config but still does not install or
  start a guest WireGuard setup script.
- `LinuxVirtualMachine/Models`: sidebar sections, USB accessory records, VM

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Afcoo/RNDIS-Tethering-VM-Passthrough](https://github.com/Afcoo/RNDIS-Tethering-VM-Passthrough) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
