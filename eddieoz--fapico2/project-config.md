---
trigger: always_on
description: Rust reimplementation of a YubiKey-class authenticator for the Raspberry Pi
---

# AGENTS.md — fapico2 firmware

Rust reimplementation of a YubiKey-class authenticator for the Raspberry Pi
Pico 2 (RP2350). Not a port of the C `pico-fido2` tree, and it does **not**
use `pico-keys-sdk` — that C SDK has no role here at all.

Most developers work on this inside a `pico/` workspace that checks out
`fapico2` alongside the reference trees listed under
[Reference implementations](#reference-implementations). Paths written as
`../<repo>` refer to those sibling checkouts; upstream URLs are given so this
file is also useful when reading the repository on its own.

---

## Read this before touching CTAP2

### 1. There are two `FidoApp`s. The one you want is usually not in `app.rs`.

```
apps/fido/src/app.rs          FidoApp<K: Keystore>   — HOST ONLY (`#[cfg(feature = "host")]`)
apps/fido/src/device_app.rs   FidoApp               — the RP2350 shell; re-exported as fapico2_fido::FidoApp
apps/fido/src/device_core.rs  the command path      — MC/GA/clientPIN/Reset/credMgmt/largeBlobs
```

The firmware runs `device_app::FidoApp` over `device_core.rs`. `app.rs` is a
**twin**, not the shipped one. Fixing only `app.rs` passes every host test and
changes nothing on hardware — that is not hypothetical; it is how the
credential-management dialect work in PR #3 went partly wrong. Change both, and
`device_core.rs` deserves the hardware proof.

`device_core.rs` is no_std but **host-compiled**, so the device path is
testable: boot a `device_app::FidoApp` with `HostTrng`/`HostSecureStore` and
drive `process_ctap2`. `apps/fido/tests/credmgmt_ctap2_spec.rs::device_twin`
does exactly that.

### 2. This firmware does NOT follow CTAP 2.1 opcodes, and that is deliberate.

`python-fido2` 2.2.1 — the library `ykman` and Yubico Authenticator are built
on — sends:

| command | fido2 2.2.1 | CTAP 2.1 spec |
|---|---|---|
| `authenticatorGetInfo` | `0x04` | `0x03` |
| `authenticatorClientPIN` | `0x06` | `0x04` |
| `authenticatorReset` | `0x07` | `0x05` |
| `authenticatorGetNextAssertion` | `0x08` | `0x06` |
| `authenticatorCredentialManagement` | `0x0A` | `0x08` |

`pico-fido`, `RS-Key` and `picoforge` share this convention. **Do not "fix" it
toward the spec** — that breaks every first-party tool and Yubico's own client.

Before diagnosing any CTAP2 mismatch, read the client's constants rather than
the spec:

```python
from fido2.ctap2.base import Ctap2;      list(Ctap2.CMD)
from fido2.ctap2.credman import CredentialManagement
print(CredentialManagement.CMD.__members__)   # GET_CREDS_METADATA=0x01, ENUMERATE_RPS_BEGIN=0x02
print(CredentialManagement.RESULT.__members__) # RP=0x03, RP_ID_HASH=0x04, TOTAL_RPS=0x05, USER=0x06 ...
```

Note the sub-commands use the **CTAP 2.0** order (metadata first), and the
response keys are **not** the spec's `rp=1, rpID=2, totalRps=7`. This "PicoForge
dialect" is what Yubico's library speaks.

### 3. Host apps read `DeviceInfo` over three interfaces, independently.

Any one failing produces a different symptom, and a green CCID test says
nothing about the other two:

| interface | command | failure symptom |
|---|---|---|
| CCID | management `READ_CONFIG` (`0x1D`) | device unseen / wrong serial |
| FIDO (CTAPHID) | `CTAP_READ_CONFIG` = frame cmd `0x42` | `CTAP2: Not supported`; Slots/Passkeys stuck |
| YubiOTP (feature reports) | `SLOT_YK4_CAPABILITIES` = `0x13` | Slots screen spins forever |

Two traps inside the FIDO path:

- The **CTAPHID INIT version bytes 13..15 carry the YubiKey firmware version**
  (5.4.0), *not* the CTAPHID protocol version. `yubikit.management.
  _ManagementCtapBackend` reads them as `device_version` and gates
  `read_device_info` on `>= 4.1`; below that, `_read_info_ctap` fabricates a
  "YubiKey 3.0 / U2F-only / no serial" record with no FIDO2 bit.
- All three must return the **same** body. They share
  `fapico2_mgmt::default_config_tlv(serial, out)`, fed from
  `platform::usb_ident::serial_hash4(chipid)` — the same value the USB
  descriptor uses.

For the YubiOTP path, return the **bare** blob: `platform::otp_hid::set_report`
already appends `!crc16(data)` (YubiKey convention, residue `0xF0B8`). Adding a
second CRC gives `BadResponseError: Invalid checksum`.

---

## Layout

```
firmware/src/
  main.rs        boot, app construction, USB device, task spawn
  boot.rs        statics (MANAGEMENT_APP, OTP_APP, FIDO_APP…), flash partitions, REBOOT
  tasks.rs       CCID task, CTAP-HID task (CTAPHID framing, presence windows)
  otp_hid.rs     YubiOTP HID frame handler (runs one INS 0x01 OTP APDU)
  ctap_hid.rs    CTAPHID assembler, CID allocator, HID command constants
  emul_main.rs   host emulation binary over TCP sockets (`--features emulation`)
  bin/{bringup,bridge,hwtest}.rs
apps/{fido,oath,openpgp,piv,mgmt,rescue,vendor_led}/
platform/src/
  dispatch.rs    the AID dispatcher every applet registers with
  usb.rs         USB device, composite interfaces, identity from PhyConfig
  otp_hid.rs     YubiOTP transport (report descriptors, feature-report state machine)
  ccid.rs, cflash.rs, cfs.rs, ckey.rs   flash / partition / key-derivation
vendor/
  opcard           OpenPGP card 3.4 (the real implementation; apps/openpgp wraps it)
  ed448-goldilocks, x448, trussed-secp256k1, trussed-brainpool
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eddieoz/fapico2](https://github.com/eddieoz/fapico2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
