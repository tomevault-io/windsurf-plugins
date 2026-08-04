---
trigger: always_on
description: Claude Code Web is making passes on this repo and **pushing commits to GitHub**,
---

# Context for Claude Code agents working on keyroost

## ⚠️ FIRST: sync with GitHub before doing any local work

Claude Code Web is making passes on this repo and **pushing commits to GitHub**,
so the GitHub remote is now the source of truth and the local checkout is
frequently behind. **Before starting any local work (and before committing),
check the remote and integrate it:**

```bash
git fetch origin
git log --oneline HEAD..origin/main   # what landed on the remote that we don't have
git status                            # branch + divergence
```

If `origin/main` (or the branch you're on) has moved ahead, **pull/rebase onto
it before writing code or committing** — do not build on a stale local tree, and
do not push a branch that diverged without reconciling first (we hit exactly
this and had to untangle a rejected push). When in doubt, stop and surface the
divergence to the user rather than committing on top of stale state.

## What this repository is

Independent, MIT/Apache-2.0 dual-licensed Rust toolchain for programming the
Token2 Molto2 / Molto2v2 programmable TOTP hardware token. Built from scratch
based on observation of the device protocol; not a fork of Token2's Python
tool. Workspace contains:

| Crate | Purpose | External deps |
|---|---|---|
| `keyroost-proto` | Pure-Rust protocol layer (SM4, SHA-1, APDU builders, MAC) | none |
| `keyroost-transport` | PC/SC reader discovery, Molto2 session, YubiKey CCID serial, OATH + OpenPGP applets | `pcsc`; `aes`/`des`/`cipher`/`getrandom`/`zeroize` (PIV mgmt-key auth); `hidapi` (non-Linux HID) |
| `keyroost-hid` | USB HID enumeration of FIDO devices via sysfs | `hidapi` (non-Linux only; Linux uses sysfs) |
| `keyroost-ctap` | FIDO2/CTAP-HID transport, CBOR, PIN protocols, credential mgmt, largeBlob | RustCrypto (`sha2`/`hmac`/`aes`/`cbc`/`p256`/`aes-gcm`), `rand_core`, `zeroize`, `miniz_oxide` (largeBlob DEFLATE); `hidapi` (non-Linux HID) |
| `keyroost-oath` | Pure-Rust Yubico/Trussed OATH (TOTP/HOTP) byte layer (APDU + TLV) | `zeroize` |
| `keyroost-openpgp` | Pure-Rust OpenPGP Card v3.4 byte layer (APDU + BER-TLV) | `zeroize` |
| `keyroost-piv` | Pure-Rust PIV (SP 800-73-4) byte layer; full management (status, GENERAL AUTHENTICATE, key-gen, cert import, PIN/PUK/mgmt-key, reset) + SPKI/PEM | `zeroize` |
| `keyroost-token2otp` | Pure-Rust Token2 OTP-on-FIDO management byte layer (APDU + HID framing, ECDH+AES seed encryption) | RustCrypto (`sha2`/`aes`/`cbc`/`p256`), `rand_core`, `zeroize` |
| `keyroost-token2prog` | Pure-Rust Token2 2nd-gen single-profile programmable-token protocol (SM4 seed/MAC, config TLV); reuses `keyroost-proto` | `zeroize` |
| `keyroost-keyring` | Friendly-name registry (`keys.json`); serial matching, no hardware | `serde`, `serde_json` |
| `keyroost-resolve` | Shared key-identity resolution (USB + CCID serials, topology match) | in-tree only |
| `keyroost-rsakey` | Host-side RSA-2048 keygen + PKCS#1/PKCS#8 (PEM/DER) loading for OpenPGP import | `rsa`, `rand`, `zeroize` (scoped exception) |
| `keyroost-import` | otpauth:// + Aegis / 2FAS / otpauth-list parsers | `zeroize`; `serde`/`serde_json` (behind `bulk`); `scrypt`/`aes-gcm`/`base64` (behind `encrypted`, for Aegis vaults) |
| `keyroost-qr` | QR 2FA import from PNG/JPEG screenshots + Google Authenticator migration batches (always built; the GUI's separate `qr` feature gates *screen capture*, not this) | `rqrr`, `png`, `jpeg-decoder`, `zeroize` |
| `keyroost-screengrab` | Windows-only GDI screen capture for QR-from-screen; the sole `unsafe` FFI crate; inert on non-Windows | `windows-sys` (Windows only) |
| `keyroost-winwebauthn` | Windows-only non-admin FIDO2 helper: detect a FIDO key, open Windows' security-key settings, relaunch elevated; inert on non-Windows | `windows-sys` (Windows only) |
| `keyroostctl` | CLI binary | `clap` (+ `clap_complete`/`clap_mangen`), `serde`/`serde_json`, `zeroize` |
| `keyroost` | egui desktop GUI | `eframe`, `egui`, `serde`/`serde_json`, `zeroize`, `base64`, plus platform UI deps (`arboard`, `rfd`, `pollster`, `png`; Linux `ashpd`/`x11rb` behind the `qr` feature) |

## Where to start reading

1. **`docs/PROTOCOL.md`** — wire format reference. APDU opcodes, the SM4-CBC
   MAC, the config TLV. Written about the device itself; doesn't reference any
   third-party implementation.
2. **`docs/BRINGUP.md`** — step-by-step plan for first-time hardware bring-up.
   This is the runbook the user wants to execute. Steps 1, 2 and 4 are
   read-only; step 3 writes a title to slot #99, step 5 writes a seed there,
   and step 6 bulk-imports into #95 onwards. Step 3 also offers a full-device
   wipe as the forgotten-key recovery path.
3. **`crates/keyroost-proto/src/`** — the protocol layer is the cleanest place
   to understand command construction. Start with `commands.rs`.

## The user's immediate goal

Program their Molto2 from a machine they control, with Claude Code running
locally so debug output and APDU hex traces can be diagnosed in-context. The
workflow during bring-up is:

1. User runs `keyroostctl --debug <subcommand>`.
2. If something looks wrong (status word other than `9000`, garbled response,
   wrong on-device behavior), agent diffs the captured hex against
   `docs/PROTOCOL.md` and edits the offset / framing in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [framefilter/keyroost](https://github.com/framefilter/keyroost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
