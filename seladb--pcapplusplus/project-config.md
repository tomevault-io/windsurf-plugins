---
trigger: always_on
description: This file provides guidance for AI coding agents (e.g. OpenAI Codex, Claude, GitHub Copilot
---

# PcapPlusPlus Development Guide

This file provides guidance for AI coding agents (e.g. OpenAI Codex, Claude, GitHub Copilot
Workspace, etc.) working inside the PcapPlusPlus repository. Read this before writing or
modifying code.

---

## Project Overview

PcapPlusPlus is a **multiplatform C++ library** for capturing, parsing, and crafting network
packets. It wraps popular packet-processing engines (libpcap, Npcap, WinPcap, DPDK, eBPF
AF_XDP, WinDivert, PF_RING) behind a clean, modern C++ API.

The codebase is organized into three libraries:

| Library    | Purpose                                                                   |
|------------|---------------------------------------------------------------------------|
| `Common++` | Shared utilities used by both Packet++ and Pcap++                         |
| `Packet++` | Protocol parsing, editing, and crafting (standalone; no libpcap required) |
| `Pcap++`   | Packet capture/send via libpcap, Npcap, DPDK, etc.                        |

---

## Repository Layout

```
PcapPlusPlus/
├── Common++/           # Common++ library source & headers
├── Packet++/           # Packet++ library source & headers
├── Pcap++/             # Pcap++ library source & headers
├── Tests/
│   ├── Packet++Test/   # Unit tests for Packet++ (protocol parsing/crafting)
│   ├── Pcap++Test/     # Unit tests for Pcap++ (live capture, file I/O, etc.)
│   └── ExamplesTest/   # Python-based tests (written with pytest) for the example applications
├── Examples/           # Example applications
│   └── Tutorials/      # Tutorial source code
├── cmake/              # CMake helper modules
└── CMakeLists.txt      # Top-level CMake build file
```

---

## Building the Project

PcapPlusPlus uses **CMake** as its build system.

### Linux / macOS

Install the prerequisite (libpcap):
```bash
# Debian/Ubuntu
sudo apt-get install libpcap-dev

# RHEL/Fedora
sudo yum install libpcap-devel

# macOS (Xcode Command Line Tools already include libpcap)
xcode-select --install
```

Configure and build:
```bash
cmake -S . -B build
cmake --build build
```

Build outputs:
- `build/Common++/libCommon++.a`
- `build/Packet++/libPacket++.a`
- `build/Pcap++/libPcap++.a`
- `build/examples_bin/` — example binaries
- `Tests/Packet++Test/Bin/Packet++Test`
- `Tests/Pcap++Test/Bin/Pcap++Test`

### Key CMake Options

| Option                            | Default | Description                                          |
|-----------------------------------|---------|------------------------------------------------------|
| `-DPCAPPP_BUILD_EXAMPLES=ON/OFF`  | `ON`    | Build example apps                                   |
| `-DPCAPPP_BUILD_TESTS=ON/OFF`     | `ON`    | Build unit tests                                     |
| `-DPCAPPP_BUILD_TUTORIALS=ON/OFF` | `OFF`   | Build tutorial binaries                              |
| `-DBUILD_SHARED_LIBS=ON/OFF`      | `OFF`   | Build shared instead of static libs                  |
| `-DPCAPPP_USE_PCAP=ON/OFF`        | `ON`    | Enable libpcap/WinPcap/Npcap support                 |
| `-DPCAPPP_USE_DPDK=ON/OFF`        | `OFF`   | Enable DPDK support                                  |
| `-DPCAPPP_USE_PF_RING=ON/OFF`     | `OFF`   | Enable PF_RING support                               |
| `-DPCAPPP_USE_XDP=ON/OFF`         | `OFF`   | Enable AF_XDP support                                |
| `-DPCAPPP_USE_WINDIVERT=ON/OFF`   | `OFF`   | Enable WinDivert support                             |
| `-DPCAPPP_BUILD_PCAPPP=ON/OFF`    | `ON`    | Build Pcap++ (turn off for Packet++ & Common++ only) |
| `-DLIGHT_PCAPNG_ZSTD=ON/OFF`      | `OFF`   | Enable Zstd PCAPNG compression                       |

### Windows (Visual Studio 2019/2022)

Download and install Npcap SDK (or WinPcap developer's pack) before configuring:
```powershell
cmake -S . -B build -G "Visual Studio 17 2022" -DPCAP_ROOT=<path_to_npcap_sdk>
cmake --build build --config Release
```

---

## Running Tests

**Always run tests from inside the test directory** — the test binaries rely on relative paths to
fixture files.

### Packet++Test (protocol parsing/crafting — no network required)

```bash
cd Tests/Packet++Test
Bin/Packet++Test
```

Run a subset by tag:
```bash
Bin/Packet++Test -t "eth;ipv4"
```

Run a specific test case:
```bash
Bin/Packet++Test -t ArpPacketCreation
```

Useful flags:

| Flag                         | Description                                                    |
|------------------------------|----------------------------------------------------------------|
| `-t / --include-tags`        | Run only tests matching the given semicolon-separated tag list |
| `-x / --exclude-tags`        | Exclude tests matching the given semicolon-separated tag list  |
| `-m / --mem-verbose`         | Verbose memory allocation output (leak debugging)              |
| `-s / --skip-mem-leak-check` | Skip the per-test memory leak check                            |

### Pcap++Test (live capture, file I/O, DPDK, etc.)

Requires `sudo` on Linux and macOS. Some tests need active network traffic on the specified interface.
```bash
cd Tests/Pcap++Test
sudo Bin/Pcap++Test -i <interface_ip>
```

To skip all tests that require live networking:
```bash
sudo Bin/Pcap++Test -n
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seladb/PcapPlusPlus](https://github.com/seladb/PcapPlusPlus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
