---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ESP32 Ethernet Router - Firmware where **WiFi STA is the uplink** (Internet) and **Ethernet is the downlink** (LAN). This is the inverse of the original ESP32 NAT Router topology. Supports NAT routing, DHCP server, VPN/WireGuard, ACL firewall, DHCP reservations, and port mapping.

**Supported hardware variants:**
- **WT32-ETH01** — ESP32 (dual-core, 240 MHz) with built-in LAN8720 Ethernet PHY
- **W5500 + ESP32-C3 SuperMini** — ESP32-C3 (single-core RISC-V, 160 MHz) with W5500 SPI Ethernet module

Both variants share all router logic. The only divergence is Ethernet MAC/PHY initialization, selected at build time via Kconfig (`CONFIG_ETH_DOWNLINK_EMAC` vs `CONFIG_ETH_DOWNLINK_W5500`).

## Build Commands

### WT32-ETH01 (ESP32 + LAN8720)
```bash
./build_firmware.sh               # Clean build → firmware/
idf.py -B build_eth_sta menuconfig
idf.py -B build_eth_sta build
idf.py -B build_eth_sta flash monitor   # 115200 bps
```

### W5500 + ESP32-C3
```bash
./build_firmware_w5500_c3.sh      # Clean build → firmware_w5500_c3/
idf.py -B build_w5500_c3 menuconfig
idf.py -B build_w5500_c3 \
  -D SDKCONFIG=sdkconfig.w5500_c3 \
  -D SDKCONFIG_DEFAULTS="sdkconfig.defaults;sdkconfig.defaults.w5500_c3" \
  build
idf.py -B build_w5500_c3 -p /dev/ttyACM0 flash monitor   # SuperMini (USB-JTAG)
idf.py -B build_w5500_c3 -p /dev/ttyUSB0 flash monitor   # DevKit-M-1 (UART)
```

**Note:** Each variant uses a separate build directory and sdkconfig file to avoid conflicts. Always pass `-D SDKCONFIG=sdkconfig.w5500_c3` for the W5500 build or it will overwrite the default `sdkconfig`.

## Architecture

### Network Topology
```
                          ESP32 Ethernet Router (WT32-ETH01)
                    ┌─────────────────────────────────────┐
                    │                                     │
Internet ──────────►│  WiFi STA (uplink)   Ethernet (LAN)│◄──── Clients
         ◄──────────│                                     │─────►
                    └─────────────────────────────────────┘
```

- **WiFi STA**: Connects to upstream AP for Internet access
- **Ethernet**: LAN interface with optional DHCP server and NAT

### Source Structure
```
main/
├── esp32_nat_router.c   # Entry point: WiFi+ETH init, event handling, LED status
├── dhcp_manager.c       # DHCP reservation management (add/del/lookup/print)
├── netif_hooks.c        # Network interface hooks for byte counting and ACL
├── portmap.c            # Port mapping table management
└── vpn_manager.c        # VPN/WireGuard configuration and management

include/
├── router_globals.h     # Global variables and shared state
├── router_config.h      # NVS namespace and config constants
├── dhcp_reservations.h  # DHCP reservation API
├── portmap.h            # Port mapping API
├── vpn_config.h         # VPN configuration structures
├── web_password.h       # Web password hashing API
└── wifi_config.h        # WiFi config parameter helpers

components/
├── acl/                 # Stateless packet filtering firewall (4 ACL lists, 16 rules each)
├── dhcpserver/          # Custom DHCP server with reservation support (overrides ESP-IDF built-in)
├── http_server/         # Web UI server (pages: /, /config, /mappings, /firewall, /vpn, /scan)
├── pcap_capture/        # PCAP packet capture with TCP streaming to Wireshark
├── remote_console/      # Network-accessible CLI via TCP (password protected)
├── cmd_router/          # CLI commands: set_sta, set_ap_ip, set_ap_dns, set_eth_nat, set_eth_dhcps, portmap, dhcp_reserve, web_ui, set_router_password, show, acl, remote_console, syslog
└── cmd_system/          # System commands: free, heap, restart, factory_reset, tasks
```

### Custom DHCP Server Component
The `components/dhcpserver/` directory contains a custom DHCP server implementation that overrides the ESP-IDF built-in version using linker wrapping (`--wrap`).

**Structure:**
```
components/dhcpserver/
├── CMakeLists.txt                 # Build config with --wrap linker flags
├── dhcpserver.c                   # Implementation (functions use __wrap_ prefix)
└── include/dhcpserver/
    ├── dhcpserver.h               # Public API
    └── dhcpserver_options.h       # DHCP option definitions
```

**How it works:**
- `CONFIG_LWIP_DHCPS` remains enabled (ESP-IDF config options work normally)
- Linker `--wrap=<func>` redirects all calls to `__wrap_<func>` implementations
- Original ESP-IDF functions available via `__real_<func>()` if needed

**Wrapped functions:**
- `dhcps_new`, `dhcps_delete`, `dhcps_start`, `dhcps_stop`
- `dhcps_option_info`, `dhcps_set_option_info`
- `dhcp_search_ip_on_mac`, `dhcps_set_new_lease_cb`
- `dhcps_dns_setserver`, `dhcps_dns_getserver` (and `_by_type` variants)

**To modify DHCP behavior:** Edit `components/dhcpserver/dhcpserver.c`

### Key Global Variables (in router_globals.h / router_config.h)
- `ssid`, `passwd` - Upstream WiFi credentials (STA)
- `static_ip`, `subnet_mask`, `gateway_addr` - Static IP config for STA
- `my_ip`, `my_ap_ip` - Current IP addresses (STA uplink, ETH downlink)
- `eth_nat_enabled` - Whether NAT is active on Ethernet (default: 1)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [martin-ger/esp32_ethernet_router](https://github.com/martin-ger/esp32_ethernet_router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
