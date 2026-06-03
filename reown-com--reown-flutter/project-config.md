---
trigger: always_on
description: This skill encodes project-specific conventions including interface-based design, freezed models, event-driven architecture, code generation patterns, and testing practices aligned with official Flutter guidelines.
---

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## AI Skills

When writing or modifying Flutter/Dart code in this repository, use the **flutter-coding** skill located at:
- `.claude/skills/flutter-coding/SKILL.md` - Core patterns, validation checklist, and Flutter/Dart best practices

This skill encodes project-specific conventions including interface-based design, freezed models, event-driven architecture, code generation patterns, and testing practices aligned with official Flutter guidelines.

## Project Overview

**Reown Flutter** is a Flutter/Dart SDK implementing the WalletConnect protocol for multi-platform applications. Reown (formerly WalletConnect) is the communications protocol for web3, bringing the ecosystem together by enabling hundreds of wallets and apps to securely connect and interact.

### Key Information
- **Repository**: https://github.com/reown-com/reown_flutter
- **Published Packages**: https://pub.dev/publishers/reown.com/packages
- **Documentation**: 
  - AppKit: https://docs.reown.com/appkit/flutter/core/installation
  - WalletKit: https://docs.reown.com/walletkit/flutter/installation
- **License**: Apache 2.0

### Tech Stack
- **Dart**: 3.8.0+ with null safety
- **Flutter**: 1.10.0+ for cross-platform UI
- **Platforms**: Android, iOS, Web, macOS, Linux, Windows
- **Cryptography**: ed25519_edwards, x25519, pinenacl, crypto, pointycastle
- **Networking**: http, web_socket_channel, connectivity_plus
- **Storage**: flutter_secure_storage, shared_preferences
- **UI Components**: flutter_svg, cached_network_image, custom_sliding_segmented_control, shimmer
- **Blockchain**: web3dart, eth_sig_util_plus, bs58
- **Code Generation**: build_runner, freezed, json_serializable
- **Testing**: flutter_test, mockito, flutter_lints

### Multi-Chain Support
- EVM chains (Ethereum, Polygon, Arbitrum, Optimism, etc.)
- Solana
- Polkadot
- Kadena
- Tron
- Cosmos

## Repository Structure

This is a **monorepo** containing multiple Flutter packages organized in a layered architecture:

```
reown_flutter/
├── packages/
│   ├── reown_core/          # Foundation layer
│   ├── reown_sign/          # Sign protocol implementation
│   ├── reown_walletkit/     # Wallet-side implementation
│   ├── reown_appkit/        # Full-featured UI toolkit
│   ├── reown_yttrium/       # Chain abstraction layer (early access)
│   ├── reown_yttrium_utils/ # Yttrium utilities
│   ├── reown_cli/           # Command-line tool
│   ├── walletconnect_pay/   # Payment protocol
│   └── pos_client/          # Point of sale SDK
├── scripts/                 # Build and utility scripts
├── .github/                 # CI/CD workflows
└── assets/                  # Shared assets
```

### Package Dependencies

The packages follow a dependency hierarchy:

**Independent packages** (no internal reown dependencies):
- **reown_yttrium**: Chain abstraction layer (ERC-4337, ERC-7702 support)
- **walletconnect_pay**: Payment protocol implementation
- **reown_cli**: Command-line tool
- **pos_client**: Point of sale SDK

**Dependent chain**:

1. **reown_core** (Foundation)
   - Depends on: `reown_yttrium`
   - Provides: networking, cryptography, storage, message serialization

2. **reown_sign** (Protocol Layer)
   - Depends on: `reown_core`
   - Provides: Sign protocol implementation, session management

3. **reown_appkit** (DApp Side)
   - Depends on: `reown_core`, `reown_sign`
   - Provides: Full UI toolkit with modals, QR codes, WebView support

4. **reown_walletkit** (Wallet Side)
   - Depends on: `reown_core`, `reown_sign`, `walletconnect_pay`
   - Provides: Wallet-side WalletConnect functionality

Note: `reown_appkit` and `reown_walletkit` are independent of each other.

### Package Details

#### reown_core
Foundation layer with:
- Networking: WebSocket via `web_socket_channel`, HTTP client
- Cryptography: ed25519, x25519, pointycastle
- Secure storage: `flutter_secure_storage` with fallback to `shared_preferences`
- Connectivity detection: `connectivity_plus`
- MessagePack serialization: `msgpack_dart`
- Pairing and session management
- Expiration tracking

#### reown_sign
Sign protocol implementation:
- Uses `web3dart` for EVM chain interactions
- `freezed` for immutable models
- Session proposal, request, and approval handling
- Auth protocol support
- Namespace management

#### reown_walletkit
Wallet implementation:
- Orchestrates `reown_core` networking and `reown_sign` protocol
- Provides wallet-side WalletConnect functionality
- Key management and signing
- Multi-chain wallet support

#### reown_appkit
Full-featured UI toolkit:
- Coinbase Wallet SDK integration
- WebView support for WalletConnect
- QR code generation (`qr_flutter_wc`)
- Comprehensive modal components
- Asset caching (`cached_network_image`)
- Shimmer loading effects

#### reown_yttrium
Chain abstraction layer (early access):
- Smart account functionality (ERC-4337, ERC-7702)
- Native platform plugins (Android/iOS)
- `freezed` for immutable models
- `eth_sig_util_plus` for signature utilities

#### walletconnect_pay
Payment protocol plugin:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reown-com/reown_flutter](https://github.com/reown-com/reown_flutter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
