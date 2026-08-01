---
trigger: always_on
description: > Quick-reference for AI agents and contributors working on this repository.
---

# AGENTS.md — canokey-core Codebase Guide

> Quick-reference for AI agents and contributors working on this repository.

---

## Project Overview

`canokey-core` is a platform-independent C11 library implementing an open-source security key. It supports:

- **FIDO2 / U2F** (CTAP2/CTAP1, Ed25519, HMAC-secret, ML-DSA-65)
- **OpenPGP Card V3.4** (RSA 2048/3072/4096, ECDSA, Ed25519, X25519)
- **PIV** (NIST SP 800-73-4, with RSA 3072/4096, Ed25519, X25519)
- **HOTP / TOTP** (OATH)
- **NDEF** (NFC tag emulation, requires NFC hardware)
- **WebUSB web interface** (browser-based configuration)
- **KBDHID / PASS** (keyboard output of OTP codes, requires touch sensor)

The library is meant to be linked by a platform port (e.g. `canokey-ciu`, `canokey-stm32`). It does **not** contain any hardware-specific code by itself.

---

## Repository Layout

```
canokey-core/
├── include/            # Public headers (device.h, apdu.h, key.h, …)
├── src/                # Platform-independent core (device.c, apdu.c, key.c, …)
├── applets/            # Per-applet implementations
│   ├── admin/          # Admin applet (config, WebUSB control)
│   ├── ctap/           # FIDO2 / CTAP2 applet
│   ├── ndef/           # NDEF / NFC tag applet
│   ├── oath/           # HOTP/TOTP applet
│   ├── openpgp/        # OpenPGP Card applet
│   ├── pass/           # KBDHID password output applet
│   └── piv/            # PIV applet
├── interfaces/
│   ├── USB/
│   │   ├── core/       # USB stack (usbd_core, usbd_ctlreq, …)
│   │   ├── device/     # USB device descriptors and canokey composite device
│   │   └── class/
│   │       ├── ctaphid/    # CTAPHID HID class driver
│   │       ├── ccid/       # CCID smart-card class driver
│   │       ├── kbdhid/     # Keyboard HID class driver
│   │       └── webusb/     # WebUSB vendor interface
│   └── NFC/            # NFC interface (FM11NC / FM11NT)
├── canokey-crypto/     # Submodule: crypto primitives (ECC, RSA, AES, SHA, …)
├── littlefs/           # Submodule: embedded filesystem
├── tinycbor/           # Submodule: CBOR encoder/decoder
├── virt-card/          # Virtual card for host-side unit/integration tests
├── test/               # CMocka unit tests
├── fuzzer/             # honggfuzz fuzzing harness
└── scripts/            # Code-generation scripts (gen_ctap_get_info.py)
```

`fido2-tests/` is referenced by CI (`.github/workflows/tests.yml`) and `test-via-pcsc/build_fido_tests.sh`. It is **not** committed to this repo (see `.gitignore`); CI checks it out into a sibling directory at runtime.

---

## Key Source Files

| File | Role |
|---|---|
| `src/device.c` | Main loop dispatch (`device_loop`), LED, touch, applet-session management |
| `src/apdu.c` | APDU parsing, chaining (`apdu_input`/`apdu_output`), shared I/O buffer |
| `src/key.c` | Key import/export, `ck_key_t` serialization, PIV/OpenPGP stream parsers |
| `src/pin.c` | PIN creation, verification, retry counter (backed by LittleFS) |
| `src/fs.c` | Thin LittleFS wrapper (`read_file`, `write_file`, `read_attr`, …) |
| `src/pke.c` | Platform key-engine buffer abstraction (hardware accelerator scratch space) |
| `interfaces/USB/class/ctaphid/ctaphid.c` | CTAPHID framing, channel management, `CTAPHID_Loop` |
| `interfaces/USB/class/ccid/ccid.c` | CCID T=1 framing, `CCID_Loop` |
| `interfaces/USB/device/usbd_desc.c` | USB descriptors (config, BOS, string) |

---

## Build System

CMake 3.16+, C11. The library target is `canokey-core`.

### Feature flags (CMake options / `-D`)

| Flag | Default | Effect |
|---|---|---|
| `ENABLE_NFC` | auto (from `HW_VARIANT`) | Compile NFC interface and NDEF applet |
| `ENABLE_APPLET_NDEF` | = `ENABLE_NFC` | Include NDEF applet |
| `ENABLE_IFACE_CTAPHID` | 1 | CTAPHID HID interface |
| `ENABLE_IFACE_CCID` | 1 | CCID interface |
| `ENABLE_IFACE_WEBUSB` | 1 | WebUSB vendor interface |
| `ENABLE_IFACE_KBDHID` | auto (`HAS_TOUCH`) | Keyboard HID interface |
| `ENABLE_PASS` | auto (`HAS_TOUCH`) | PASS/KBDHID OTP applet |
| `ENABLE_DEBUG_OUTPUT` | ON | `DBG_MSG`/`ERR_MSG` via `printf` |
| `ENABLE_BYPASS_USER_PRESENCE` | OFF | Skip all touch checks (testing only) |
| `ENABLE_TESTS` | OFF | Build CMocka unit tests + virt-card |
| `ENABLE_FUZZING` | OFF | Build honggfuzz harness |
| `VIRTCARD` | OFF | Build only the virtual-card targets |

### Running unit tests

```bash
mkdir build && cd build
cmake .. -DENABLE_TESTS=ON -DCMAKE_BUILD_TYPE=Debug
make -j$(nproc)
ctest --output-on-failure
```

---

## Platform Porting Contract

A platform port **must** implement every symbol declared in `include/device.h`:

### Mandatory

```c
void     device_delay(int ms);
uint32_t device_get_tick(void);            // millisecond tick counter
int      device_spinlock_lock(volatile uint32_t *lock, uint32_t blocking);
void     device_spinlock_unlock(volatile uint32_t *lock);
int      device_atomic_compare_and_swap(volatile uint32_t *var, uint32_t expect, uint32_t update);
void     led_on(void);
void     led_off(void);
void     device_set_timeout(void (*callback)(void), uint16_t timeout); // hardware timer with IRQ
```

### Mandatory for NFC (FM11NC SPI)

```c
void fm_csn_low(void);
void fm_csn_high(void);
void spi_transmit(const uint8_t *buf, uint8_t len);
void spi_receive(uint8_t *buf, uint8_t len);
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [canokeys/canokey-core](https://github.com/canokeys/canokey-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
