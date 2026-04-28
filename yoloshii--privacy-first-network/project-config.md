---
trigger: always_on
description: You are assisting a user in deploying a network-wide VPN privacy router. This document provides your operational framework.
---

# Agent Instructions: Privacy Router Deployment

You are assisting a user in deploying a network-wide VPN privacy router. This document provides your operational framework.

> **Development Note:** This stack was developed, tested, and deployed using **Claude Opus 4.6** via Claude Code with **Mullvad VPN**. The methodology below reflects lessons learned during that implementation, including 14 production issues resolved through iterative debugging.

## Your Mission

Help the user deploy a privacy router that:
1. Routes ALL network traffic through an encrypted VPN tunnel
2. Implements a firewall-based kill switch (no traffic leaks)
3. Automatically recovers from failures
4. **Optional:** DNS-level ad/tracker blocking (AdGuard Home)
5. **Optional:** Threat intelligence blocking (BanIP)

---

## Information Discovery Hierarchy

**All placeholder values in examples (YOUR_*, VPN_PROVIDER_*, etc.) must be resolved. Use this priority order:**

### 1. Local Discovery (Preferred)
Probe the user's environment automatically where possible:
```
- Network topology: ip addr, ip route, /etc/config/network
- Interface names: ip link, ls /sys/class/net/
- Current gateway/DNS: cat /etc/resolv.conf, ip route show default
- DHCP leases: cat /tmp/dhcp.leases
- Firewall state: iptables -L -n, uci show firewall
- Existing configs: /etc/amneziawg/, /etc/config/
```

### 2. User Input (Secrets Only)
**When a secret is needed, always ask for user consent first.**

The primary secret is the **VPN private key** from the user's provider account.

**Consent flow (when deployment requires a secret):**

1. **Agent asks permission:** "I need your VPN private key to configure the tunnel. How would you like to proceed?"

2. **User chooses one of:**
   - **Option A:** "I've put it in `secrets.env`" → Agent reads from file
   - **Option B:** "Here it is: [key]" → User pastes directly in chat
   - **Option C:** "I'll enter it manually" → Agent provides file path and field name

3. **Agent proceeds** only after user consents and provides the secret via their chosen method

**Option A: secrets.env (convenience file)**

For users who prefer to prepare secrets in advance:

```bash
cp secrets.env.example secrets.env
nano secrets.env  # Fill in VPN_PRIVATE_KEY
```

Contents:
```
VPN_PRIVATE_KEY=           # Required - from VPN provider account
# ROUTER_ADMIN_PASSWORD=   # Optional - OpenWrt admin
# ADGUARD_ADMIN_PASSWORD=  # Optional - AdGuard admin
```

> **Note:** This is NOT a traditional `.env` file loaded into app environment variables.
> It's a **secrets reference file** - the agent reads values during deployment and
> injects them into configuration files (e.g., `awg0.conf`). No runtime environment
> variables are involved. The file is gitignored - secrets never leave user's machine.

**Option C: Manual entry**

If user declines agent handling, provide clear instructions:
```
File: /etc/amneziawg/awg0.conf
Field: PrivateKey = <paste your key here>
```

Do NOT prompt for values that can be discovered or researched (IPs, public keys, endpoints).

### 3. Deep Research (Current/External Values)
Use web research for values requiring up-to-date discovery:
```
- VPN server endpoints (IPs, ports, public keys)
- VPN provider DNS IPs (may change)
- Provider-specific configuration requirements
- Current best practices and security advisories
```

**Examples of what to discover vs. ask:**

| Value | Discovery Method |
|-------|-----------------|
| WAN interface name | `ip link` or `uci show network` |
| LAN IP scheme | `ip addr show br-lan` |
| AdGuard IP | Check DHCP config or ask user preference |
| Mullvad server IP | Research current server list |
| Mullvad public key | Research or download from provider |
| User's private key | **ASK** - this is a secret |
| VPN provider DNS | Research provider documentation |

**The goal:** Minimize user prompts. Discover what's discoverable, research what's public, ask only for secrets (with user consent).

---

## Phase 1: Network Audit

**Before any implementation, gather this information:**

### 1.1 Current Network Topology

Ask the user or probe their system to determine:

```
□ ISP connection type (fiber ONT, cable modem, DSL, LTE/5G)
□ Current router model and capabilities
□ Current IP scheme (e.g., 192.168.1.0/24)
□ Number and types of devices on network
□ Any existing VLANs or network segmentation
□ Static IP assignments that must be preserved
□ DHCP reservations to migrate (see below)
```

**DHCP Reservations (User Decision):**

Ask the user which devices should keep their current IP addresses. Migrating DHCP reservations avoids reconfiguring:
- Smart home devices and their apps
- Network printers and scanner configs
- Media servers, NAS shares, and bookmarks
- Port forwarding rules and firewall exceptions
- Any scripts or configs with hardcoded IPs

**How to collect:**
```bash
# On existing router (OpenWrt)
cat /tmp/dhcp.leases
uci show dhcp | grep host

# On existing router (consumer routers)
# Check admin panel → DHCP → Reserved/Static leases
```

**Migration approach:**
1. Export current reservations (MAC → IP mappings)
2. After privacy router deployment, recreate in new DHCP config

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yoloshii/privacy-first-network](https://github.com/yoloshii/privacy-first-network) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
