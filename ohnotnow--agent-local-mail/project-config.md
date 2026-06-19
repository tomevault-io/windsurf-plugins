---
trigger: always_on
description: >
---


# agent-mail — a LAN mail server for AI agents

This skill installs **maddy** (a single-binary Go mail server) on a Linux
host and configures it for LAN-only multi-user delivery with a self-signed
TLS cert. The default use case is "let my locally-running AI agent send me
an email" via a mail client like Apple Mail, but it works for any internal
mail scenario.

The reference deployment is a Raspberry Pi reached over SSH (`<target>`),
with mail state on a USB SSD to spare the SD card. The same recipe works
on any Debian/Ubuntu host with `systemd`, `curl`, `tar`, and `zstd`. A
plain-text variant is documented at the end for anyone who genuinely
doesn't want TLS.

## Before you run

Confirm these with the user:

1. **Target host** — SSH alias, e.g. `elf`. Passwordless `sudo` should be
   available (the skill checks).
2. **Mail domain** — what comes after the `@`. The convention is
   `<hostname>.local` (e.g. `magicelf.local`) so that the existing avahi/
   Bonjour `.local` resolution lets every device on the LAN find the
   server without extra DNS config.
3. **State directory** — where mail and the auth DB live. On a Pi this
   should be a path on the SSD/USB (e.g. `/mnt/ssd/maddy`) to spare the
   SD card. Default `/var/lib/maddy` is fine on a non-Pi box.
4. **Accounts to create** — at minimum **one for the user themselves**
   (e.g. `me`, or whatever local-part they like) plus one per agent
   (e.g. `hermes`, `claude`). The user mailbox is what their mail client
   (Apple Mail etc.) logs into to *be* a participant on the LAN — without
   it they'd have to impersonate an agent account, which gets conceptually
   confusing fast. Don't suggest names yourself from the harness context
   (their home directory, their email) — ask.
5. **TLS or plain?** — Default is self-signed TLS, because most modern
   clients (Apple Mail, Outlook, Python's `imaplib` used by Hermes) will
   refuse a plaintext connection or warn obnoxiously. To go fully plain,
   replace the `tls file ...` line in `maddy.conf` with `tls off` and
   drop the `tls://0.0.0.0:993` listener — see the alternate config in
   the "Plain-text mode" section below.

Don't agonise over the domain — it's just a config edit to change later, as
long as no real mail has been received yet.

## Steps

### 1. Recon the target

Over SSH, gather:

```bash
ssh <target> '
  cat /etc/os-release | head -3
  uname -m                                # need aarch64 or x86_64
  hostname
  df -h /                                  # SD card free space
  df -h <state-dir-parent> 2>/dev/null     # SSD free space if not root
  sudo ss -tlnp | grep -E ":(25|143|587|465|993) " || echo "mail ports free"
  dpkg -l | grep -iE "postfix|dovecot|exim|maddy" || echo "no existing mail software"
  systemctl is-active avahi-daemon || echo "no avahi"
'
```

Confirm:

- Architecture is `aarch64` or `x86_64` (maddy ships prebuilt binaries for both).
- Mail ports 25/143/587 are not already taken by Postfix/Exim/Dovecot. On
  Debian, **exim4 is installed by default and binds `127.0.0.1:25`** — this
  must be stopped and disabled before maddy can start.
- Avahi is running (otherwise `.local` won't resolve from LAN clients —
  install `avahi-daemon` or use the host's IP).

### 2. Disable any conflicting mail daemon

```bash
ssh <target> 'sudo systemctl stop exim4 && sudo systemctl disable exim4'
```

(Substitute `postfix`, `dovecot`, etc. if those were found instead. Don't
uninstall — disabling is reversible.)

### 3. Download and install maddy

Find the latest release via the GitHub API. Pick the asset matching the
target arch — `aarch64-linux-musl.tar.zst` for Pi, `x86_64-linux-musl.tar.zst`
for amd64.

```bash
TAG=$(curl -s https://api.github.com/repos/foxcpp/maddy/releases/latest | \
  grep '"tag_name"' | cut -d'"' -f4)
ARCH=$(ssh <target> 'uname -m')   # aarch64 or x86_64
URL="https://github.com/foxcpp/maddy/releases/download/${TAG}/maddy-${TAG#v}-${ARCH}-linux-musl.tar.zst"

ssh <target> "
  set -e
  cd /tmp
  curl -sSL -o maddy.tar.zst '$URL'
  rm -rf /tmp/maddy-extract && mkdir /tmp/maddy-extract
  tar --zstd -xf maddy.tar.zst -C /tmp/maddy-extract
"
```

On the host:

```bash
ssh <target> '
  set -e
  SRC=$(ls -d /tmp/maddy-extract/maddy-*)
  sudo install -m 0755 $SRC/maddy /usr/local/bin/maddy
  getent group maddy >/dev/null || sudo groupadd -r maddy
  id maddy >/dev/null 2>&1 || sudo useradd -r -g maddy -d /var/lib/maddy -s /usr/sbin/nologin maddy
  sudo mkdir -p <state-dir>
  sudo chown maddy:maddy <state-dir>
  sudo chmod 0750 <state-dir>
  sudo mkdir -p /etc/maddy /etc/systemd/system/maddy.service.d
  sudo install -m 0644 $SRC/systemd/maddy.service /etc/systemd/system/maddy.service
'
```

### 3a. Generate a self-signed TLS cert (skip if going plain-text)

Most clients won't talk plaintext. Generate a self-signed cert with SANs
covering every hostname the server is reachable as — LAN mDNS name,
short hostname, LAN IP, plus the Tailscale name and IP if Tailscale is
installed.

```bash
# Detect optional Tailscale identity
TS_FQDN=$(ssh <target> 'tailscale status --json 2>/dev/null | python3 -c "import json,sys;d=json.load(sys.stdin);print(d[\"Self\"][\"DNSName\"].rstrip(\".\"))"' 2>/dev/null || true)
TS_IP=$(ssh <target> 'tailscale ip -4 2>/dev/null' || true)
LAN_IP=$(ssh <target> "hostname -I | awk '{print \$1}'")


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ohnotnow/agent-local-mail](https://github.com/ohnotnow/agent-local-mail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
