---
trigger: always_on
description: > minimal Debian 13 desktop with GNOME and Wayland
---

# debgno

> minimal Debian 13 desktop with GNOME and Wayland

- every package is an explicit choice

## Architecture

All installer files are generated from TypeScript source:

```
src/
├── distro.gen.ts       # Orchestrator — computes hashes, outputs all generated files
├── config.ts          # Constants, package/verify lists, download manifest, shared paths, DebgnoConfig type
├── preseed.ts         # generate_preseed({base_url, hashes}) — late_command from DOWNLOAD_FILES
├── post_install.ts    # generate_post_install({mozilla_key_hash, grub_timeout})
├── verify.ts          # generate_verify({mozilla_key_hash, grub_timeout})
├── setup.ts           # generate_setup_nvidia() + generate_setup_tx()
└── mozilla.ts         # generate_mozilla_sources() + generate_mozilla_pin() + generate_policies_json()
```

**Generated files** in `distro/` (committed, do not edit directly):

```
distro/
├── preseed.cfg          # Debian installer automation (GitHub raw URLs + SHA256 verification)
├── preseed-local.cfg    # Local testing variant (10.0.2.2:8000 URLs, gitignored)
├── post-install.sh      # Base system setup (fetched via network during install)
├── debgno-setup-nvidia.sh # Optional NVIDIA driver setup (user runs post-install)
├── debgno-setup-tx.sh     # Optional tx install for dev environment provisioning
├── debgno-verify.sh       # Automated post-install verification
├── mozilla.sources      # deb822 APT source for Mozilla
├── mozilla-pin          # APT pin for Firefox packages
└── policies.json        # Firefox policy configuration
```

**Shared constants** in `config.ts`:

- `BASE_URL` / `LOCAL_URL` — production and QEMU test URL bases
- `MOZILLA_KEY_HASH` — shared between post-install.sh and debgno-verify.sh
- `GRUB_TIMEOUT` — shared between post-install.sh and debgno-verify.sh
- `PACKAGES` — packages installed during post-install, verified by debgno-verify.sh
- `UNWANTED_PACKAGES` — blocklist checked by debgno-verify.sh
- `VERIFY_EXTRA_PACKAGES` — expected dependencies verified but not explicitly installed (e.g., gnome-shell)
- `DOWNLOAD_FILES` — manifest of files downloaded during install (drives preseed late_command)
- `STAGING_DIR` — temp directory for files fetched by late_command, cleaned up after post-install
- `POST_INSTALL_LOG`, `POST_INSTALL_MARKER`, `MOZILLA_KEY_PATH`, `FIREFOX_POLICIES_PATH` — paths shared between post-install.sh and debgno-verify.sh

**SHA256 integrity**: `distro.gen.ts` computes hashes of all files in `DOWNLOAD_FILES` and bakes
them into preseed.cfg's late_command. Each downloaded file is verified before execution. If a hash
doesn't match, the install aborts.

## Install Flow

The Debian installer (d-i) loads preseed.cfg, which answers installer questions and defines a
`late_command` that runs after packages are installed but before first boot.

```
1. User boots netinst ISO, enters preseed URL at GRUB
2. d-i loads preseed.cfg, runs automated install
3. late_command executes (installer environment, NOT the target system):
   a. wget downloads all files in DOWNLOAD_FILES manifest
   b. Each file is SHA256-verified against hashes baked into preseed.cfg
   c. `in-target /tmp/post-install.sh` runs post-install inside the target chroot
4. post-install.sh (inside chroot):
   a. Installs GNOME, audio, security packages
   b. Sets up Mozilla APT repo + Firefox (soft failure — doesn't block install)
   c. Configures firewall, DNS-over-TLS, zram swap, GRUB timeout
   d. Writes /var/lib/post-install-complete marker
5. First boot → GDM login → GNOME Wayland session
```

**Two preseed variants**: `preseed.cfg` (production, GitHub URLs) and `preseed-local.cfg`
(testing, 10.0.2.2:8000 URLs). Both are generated from the same `preseed.ts` with different
`base_url` values. The downloaded files and their hashes are identical.

**Key distinction**: `late_command` runs in the installer environment where files are at
`/target/...`. `in-target` runs commands inside the chroot where `/target` becomes `/`.
Files downloaded to `/target/tmp/` are accessible as `/tmp/` inside `in-target`.

## Shell Scripting Constraints

The generated shell scripts run inside a Debian installer chroot, which has limitations:

- **No `pipefail`** — triggers [Debian bug #969949](https://bugs-devel.debian.org/969949) with ucf. Use `set -eu` only.
- **No process substitution** — `exec > >(tee ...)` breaks in chroot. Use simple `exec >` redirect.
- **No CD-ROM apt source** — must `sed` remove before `apt-get update` (not accessible in chroot).
- **DEBIAN_FRONTEND=noninteractive** — required to prevent interactive prompts during `apt-get install`.
- **Busybox utilities** — the d-i environment uses busybox, not GNU coreutils. Use short flags only (e.g., `sha256sum -c -` not `sha256sum -c --quiet -`).

**Post-boot scripts** (`debgno-setup-nvidia.sh`, `debgno-setup-tx.sh`) run on the fully installed system, not
in the chroot, so they can use `set -euo pipefail` and other bash features safely.

## Workflow

1. Edit source files in `src/`
2. Run `gro gen` to regenerate all output files
3. Verify with `gro gen --check`
4. Commit both source and generated output

**Never edit generated files directly** — changes will be overwritten by `gro gen`.

## Security


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ryanatkn/debgno](https://github.com/ryanatkn/debgno) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
