---
trigger: always_on
description: A standalone, minimal, auto-updating NixOS configuration for laptops/desktops.
---

# notenix — Portable NixOS GNOME Desktop Flake

A standalone, minimal, auto-updating NixOS configuration for laptops/desktops.
All NixOS modules live in this repo under the `notenix.*` option namespace.
No external module frameworks — nixpkgs and disko are the only flake inputs.

## Repo structure

```
flake.nix               — nixosConfigurations (notenix, vm-headless, vm-gnome) + install package
modules/                — all NixOS option modules, imported as nixosModules.default
  default.nix           — imports all category modules below
  system/
    install.nix         — notenix.system.install.* (hostname, user, locale, keyboard)
    nix.nix             — notenix.system.nix.* (flakes, GC, unfree, fast shutdown)
    autoupgrade.nix     — notenix.system.autoupgrade.* (daily flake rebuild + notify)
  boot/
    systemd-boot.nix    — notenix.boot.systemd-boot.* (EFI boot, kernel)
  desktop/
    gnome.nix           — notenix.desktop.gnome.* (GNOME, GDM, extensions, dconf)
  applications/
    flatpak.nix         — notenix.applications.flatpak.* (Flathub, package list)
  network/
    networkmanager.nix  — notenix.network.networkmanager.*
  hardware/
    bluetooth.nix       — notenix.hardware.bluetooth.*
    printing.nix        — notenix.hardware.printing.*
    sound.nix           — notenix.hardware.sound.*
  security/
    sudo.nix            — notenix.security.sudo.wheelNeedsPassword
hosts/notenix/
  configuration.nix     — reference host; all notenix.* options for the machine
  disk.nix              — disko disk layout
_files/                 — helper scripts (notify-users.sh)
```

## Flake inputs

| Input | Purpose |
|-------|---------|
| `nixpkgs` | nixos-25.11 |
| `disko` | disk partitioning for install |

## Option namespace

All module options live under `notenix.*`. Example:

```nix
notenix.system.install = {
  enable          = true;
  hostName        = "mymachine";
  userName        = "youruser";
  userDescription = "Your Name";
  timeZone        = "Europe/Ljubljana";
  locale          = "sl_SI.UTF-8";
  keyboardLayout  = "si";
};
notenix.boot.systemd-boot.enable        = true;
notenix.desktop.gnome.enable            = true;
notenix.applications.flatpak.enable     = true;
notenix.system.nix.enable               = true;
notenix.system.autoupgrade.enable       = true;
notenix.system.autoupgrade.flakeRepo    = "github:yourusername/yourrepo";
notenix.network.networkmanager.enable   = true;
notenix.hardware.bluetooth.enable       = true;
notenix.hardware.printing.enable        = true;
notenix.hardware.sound.enable           = true;
```

## nixosConfigurations

| Name | Purpose |
|------|---------|
| `notenix` | Reference configuration for the real laptop; used by `nixos-rebuild` |
| `vm-headless` | Minimal headless VM for smoke-testing (user: `user` / pass: `notenix`) |
| `vm-gnome` | Full GNOME desktop VM for visual/interactive testing |

## Service debugging (small-first)

Use shortest reproducible path.

1. Locate option + module
  - Check host `hosts/<name>/configuration.nix`.
  - Check module under `modules/**` for option behavior.
2. Check runtime unit
  - `systemctl status <unit>`
  - `journalctl -u <unit> -n 200 --no-pager`
3. Check dependency chain
  - Timers, sockets, targets, reverse proxy (if used).
4. Check app-native logs/tools
  - Prefer native CLI status command when available.
5. Close minimal
  - Cause, smallest fix, single verify command.

Run VMs:
```bash
nix run .#vm          # headless
nix run .#vm-gnome    # GNOME desktop (needs QEMU display)
```

## Adding a new host

1. Copy `hosts/notenix/` to `hosts/<yourhostname>/`
2. Edit `hosts/<yourhostname>/configuration.nix` — update identity and module options
3. Register in `flake.nix` under `nixosConfigurations`:
   ```nix
   <yourhostname> = lib.nixosSystem {
     inherit system;
     modules = [
       self.nixosModules.default
       disko.nixosModules.disko
       ./hosts/<yourhostname>/configuration.nix
       ./hosts/<yourhostname>/disk.nix
     ];
   };
   ```

## Adding a feature flag

`default.yaml` is the **single source of truth** for all dynamic kanal tabs (features, extensions, apps). `constants.py`, `cli.py`, and `gui/window.py` all read it at runtime — no Python changes needed for ordinary features or extensions.

### 1. `pkgs/kanal/src/kanal/default.yaml`

**Bool feature** (toggle in the Features tab):
```yaml
# under tabs → id: features → items:
- key: notenix.features.myFeature
  id: myFeature
  const: MY_FEATURE          # becomes KEY_FEATURE_MY_FEATURE constant auto-magically
  title: My Feature
  subtitle: "One-line description shown in the GUI"
  default: false
```

Optional `extra` field wires a bool feature to also add/remove an entry from the GNOME extensions list:
```yaml
  extra:
    type: gnome_extension
    value: tailscale-status   # extension id in notenix.desktop.gnome.extensions
```

**List extension** (toggle in the Extensions tab):
```yaml
# under tabs → id: extensions → items:
- id: my-extension-id        # matches the _extPkgs key in gnome.nix
  title: My Extension
  subtitle: "What it does"
  default: false
```

**List extension with selectable package source** (source picker shown when Experimental is ON):
```yaml
- id: my-extension-id
  title: My Extension

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [n1x05/notenix](https://github.com/n1x05/notenix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
