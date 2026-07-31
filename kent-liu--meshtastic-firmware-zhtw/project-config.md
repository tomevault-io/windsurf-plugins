---
trigger: always_on
description: This document provides context and guidelines for AI assistants working with the Meshtastic firmware codebase.
---

# Meshtastic Firmware - Copilot Instructions

This document provides context and guidelines for AI assistants working with the Meshtastic firmware codebase.

## Project Overview

Meshtastic is an open-source LoRa mesh networking project for long-range, low-power communication without relying on internet or cellular infrastructure. The firmware enables text messaging, location sharing, and telemetry over a decentralized mesh network. The project uses **C++17** as its language standard across all platforms.

### Supported Hardware Platforms

- **ESP32** (ESP32, ESP32-S3, ESP32-C3, ESP32-C6) - Most common platform
- **nRF52** (nRF52840, nRF52833) - Low power Nordic chips
- **RP2040/RP2350** - Raspberry Pi Pico variants
- **STM32WL** - STM32 with integrated LoRa
- **Linux/Portduino** - Native Linux builds (Raspberry Pi, etc.)
- **macOS native** - Headless `meshtasticd` on Apple Silicon / x86_64; see `variants/native/portduino/platformio.ini` for Homebrew prereqs + CH341 LoRa setup

### Supported Radio Chips

- **SX1262/SX1268** - Sub-GHz LoRa (868/915 MHz regions)
- **SX1280** - 2.4 GHz LoRa
- **LR1110/LR1120/LR1121** - Wideband radios (sub-GHz and 2.4 GHz capable, but not simultaneously)
- **RF95** - Legacy RFM95 modules
- **LLCC68** - Low-cost LoRa

### MQTT Integration

MQTT provides a bridge between Meshtastic mesh networks and the internet, enabling nodes with network connectivity to share messages with remote meshes or external services.

#### Key Components

- **`src/mqtt/MQTT.cpp`** - Main MQTT client singleton, handles connection and message routing
- **`src/mqtt/ServiceEnvelope.cpp`** - Protobuf wrapper for mesh packets sent over MQTT
- **`moduleConfig.mqtt`** - MQTT module configuration

#### MQTT Topic Structure

Messages are published/subscribed using a hierarchical topic format:

```
{root}/{channel_id}/{gateway_id}
```

- `root` - Configurable prefix (default: `msh`)
- `channel_id` - Channel name/identifier
- `gateway_id` - Node ID of the publishing gateway

#### Configuration Defaults (from `Default.h`)

```cpp
#define default_mqtt_address "mqtt.meshtastic.org"
#define default_mqtt_username "meshdev"
#define default_mqtt_password "large4cats"
#define default_mqtt_root "msh"
#define default_mqtt_encryption_enabled true
#define default_mqtt_tls_enabled false
```

#### Key Concepts

- **Uplink** - Mesh packets sent TO the MQTT broker (controlled by `uplink_enabled` per channel)
- **Downlink** - MQTT messages received and injected INTO the mesh (controlled by `downlink_enabled` per channel)
- **Encryption** - When `encryption_enabled` is true, only encrypted packets are sent; plaintext JSON is disabled
- **ServiceEnvelope** - Protobuf wrapper containing packet + channel_id + gateway_id for routing
- **JSON Support** - Optional JSON encoding for integration with external systems (disabled on nRF52 by default)

#### PKI Messages

PKI (Public Key Infrastructure) messages have special handling:

- Accepted on a special "PKI" channel
- Allow encrypted DMs between nodes that discovered each other on downlink-enabled channels

## Encryption & Key Management

Meshtastic packets on the air are typically encrypted one of two ways: the **per-channel symmetric** layer (AES-CTR with a shared PSK) for broadcasts and channel traffic, and the **per-peer PKI** layer (X25519 ECDH → AES-256-CCM) for direct messages and remote admin. A channel with a 0-byte PSK (or Ham mode, which wipes PSKs) transmits cleartext — see the size table below. Both are implemented in `src/mesh/CryptoEngine.cpp`; the send/receive dispatch lives in `src/mesh/Router.cpp`; admin authorization lives in `src/modules/AdminModule.cpp`.

### High-level model

- **Channels** are symmetric rooms: anyone with the PSK can read any message on the channel. Channel 0 is the "primary" channel and ships with the short-form default PSK on factory devices, forming the public mesh most users join. (The LoRa modem preset `LONG_FAST` lives on `config.lora.modem_preset` and is an independent field — don't conflate "channel 0 default PSK" with the modem preset name.)
- **DMs** addressed to a single node require PKI so that other holders of the channel PSK can't read them. Outside Ham mode, Meshtastic does not fall back to channel-symmetric encryption when the destination public key is unknown.
- **Remote admin** is a DM carrying an `AdminMessage`. The receiver only acts on it if the sender's public key is on its allowlist (`config.security.admin_key[0..2]`).
- **Ham mode** (`owner.is_licensed=true`, where `owner` is the local `meshtastic_User` record) disables PKI entirely and sends cleartext — FCC Part 97 prohibits encryption on amateur bands.
- **No ratchet, no session.** Every packet is encrypted from scratch — a stateless design that matches the high-loss, store-and-forward nature of LoRa.

### Symmetric channel encryption (AES-CTR)

`CryptoEngine::encryptPacket` / `decrypt` / `encryptAESCtr` in `src/mesh/CryptoEngine.cpp`.

- **Cipher**: AES-CTR, AES-128 or AES-256 depending on key length. Same routine in both directions (CTR is a stream cipher, so encrypt == decrypt).
- **Key**: `ChannelSettings.psk` bytes. Size semantics:
  - **0 bytes** → no encryption, cleartext on the air

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kent-Liu/meshtastic-firmware-zhtw](https://github.com/Kent-Liu/meshtastic-firmware-zhtw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
