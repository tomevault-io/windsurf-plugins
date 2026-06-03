---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

luci-app-fleth is a LuCI-based web interface for OpenWrt that automatically configures IPv4 over IPv6 tunneling in Japan. It supports DS-Lite, MAP-E, and IPIP6H (Independent IP) protocols and automatically detects ISP providers and regional configurations (East/West Japan).

### Key Features

- **Automatic ISP Detection**: Identifies DS-Lite and MAP-E providers via DNS/connectivity tests
- **Regional Configuration**: Auto-detects East/West Japan regions
- **Protocol Support**: DS-Lite, MAP-E, IPIP6H (Independent IP tunnel)
- **IPv6 Mode Auto-Configuration**: Detects /56 (PD) or /64 (SLAAC) and configures accordingly
- **Prefix Alignment Checking**: Validates IPv6 prefix alignment for MAP-E/IPIP6 tunnels
- **Port Highlighting**: Visual highlighting of "special" ports (palindromes, repeating digits, etc.)
- **map.sh Patching**: Fixes OpenWrt MAP-E bugs (only first port group working, broken ICMP)
- **Pending State Detection**: Recognizes newly constructed fiber connections awaiting ISP setup
- **SHA256 Verification**: Security checks for downloaded patches
- **Port Forward Hook**: Integration with LuCI firewall port forwarding page

## Build Commands

### OpenWrt Package Build
```bash
# Build within OpenWrt build environment
git clone https://github.com/makeding/luci-app-fleth package/huggy/luci-app-fleth
make package/huggy/luci-app-fleth/compile
```

### Internationalization
```bash
# Compile translation files (Japanese, Simplified Chinese, Traditional Chinese)
po2lmo po/ja/fleth.po root/usr/lib/lua/luci/i18n/fleth.ja.lmo
po2lmo po/zh_Hans/fleth.po root/usr/lib/lua/luci/i18n/fleth.zh_Hans.lmo
po2lmo po/zh_Hant/fleth.po root/usr/lib/lua/luci/i18n/fleth.zh_Hant.lmo
```

## Architecture

### Core Components

1. **Frontend (LuCI Interface)**
   - `htdocs/luci-static/resources/view/fleth.js` (501 lines)
     - Main web interface with 3 tabs: Information, General Settings, Tools
     - Real-time status display (Area, Prefix Length, DS-Lite/MAP-E provider)
     - Port highlighting with caching (`_portHighlightCache`)
     - IPv6 mode configuration buttons (SLAAC/PD)
     - map.sh patch management UI
   - `htdocs/luci-static/resources/view/fleth-hook.js`
     - Injects port availability info into LuCI's port forwarding page
     - Shows available MAP-E port ranges when configuring firewall rules

2. **Backend Shell Script**
   - `root/usr/sbin/fleth` (694 lines) - Main configuration engine
     - **Area Detection** (`get_area`, line 51-75): Uses DNS resolution and IPv6 connectivity tests
     - **DS-Lite Provider Detection** (`get_dslite_provider`, line 325-353):
       - transix: Only resolvable from within ISP network
       - xpass: Resolvable but only pingable from within network
       - v6connect: Only resolvable from within ISP network
     - **MAP-E Provider Detection** (`get_mape_provider`, line 297-322): Calls Lua calculation engine
     - **Pending State Detection** (`get_pending_status`, line 77-95): Checks IPv6 prefix ranges
     - **UCI Configuration** (`set_interface`, line 355-446): Writes tunnel settings
     - **IPv6 Mode Setup** (`setup_ipv6_slaac`, `setup_ipv6_pd`, line 507-599):
       - SLAAC (/64): For 1Gbps plans without Hikari Denwa
       - PD (/56): For 10Gbps plans or with Hikari Denwa
     - **Prefix Alignment Check** (`check_prefix_alignment`, line 181-231): Validates 4th hextet ends with '00'
     - **map.sh Patching** (`patch_map.sh`, `restore_map.sh`, line 600-656): Downloads and verifies fixed version
     - **Concurrent Execution Control** (line 449-451): Simple process counting (TODO: use flock)

3. **MAP-E Calculation Engine**
   - `root/usr/sbin/fleth-map-e.lua` (984 lines)
     - Implements bitwise operations (band, bor, lshift, rshift) for Lua 5.1
     - Contains MAP-E rule tables for JPNE, BIGLOBE, OCN, NURO providers
     - Calculates: peeraddr, ipaddr, prefix, PSID, offset, available ports
     - Three rule sets: `ruleprefix31`, `ruleprefix38`, `ruleprefix38_20`
     - NOTE: Duplicate `band` function definitions (line 4 and 17) - can be cleaned up

4. **IPIP6H Protocol Handler**
   - `root/lib/netifd/proto/ipip6h.sh` (196 lines)
     - Custom netifd protocol for Independent IP tunnels
     - DNS resolution of peer address with retry
     - IPv6 prefix delegation support with interface_id
     - Prefix alignment validation (calls `fleth check_alignment`)
     - Dynamic interface creation for static IPv6 addressing
     - Comprehensive logging for troubleshooting

5. **System Integration**
   - `root/etc/init.d/fleth` (17 lines)
     - System service daemon (START=99)
     - Runs `fleth auto` on start/reload
   - `root/etc/hotplug.d/iface/70-fleth` (73 lines)
     - Triggers `fleth auto` when uplink interface comes up
     - Tunnel activation via delayed ping (30s) for map/dslite/ipip6/ipip6h protocols
     - Process management with PID files to prevent duplicate activations
   - `root/etc/uci-defaults/luci-app-fleth`
     - Sets default configuration on first install

### Configuration Management

Uses OpenWrt's UCI (Unified Configuration Interface):

```bash
# Main configuration section: fleth.global

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [makeding/luci-app-fleth](https://github.com/makeding/luci-app-fleth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
