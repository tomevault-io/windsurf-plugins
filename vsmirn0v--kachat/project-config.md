---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

KaChat is a native iOS messaging and payment application built on the Kaspa blockchain. It enables encrypted peer-to-peer messaging with on-chain payments using SwiftUI.

## Build Commands

Important, never build yourself, always ask user to build.

```bash
# Open project in Xcode
open KaChat.xcodeproj

# Build from command line
xcodebuild -project KaChat.xcodeproj -scheme KaChat -destination 'platform=iOS Simulator,name=iPhone 17' build

# Run tests
xcodebuild -project KaChat.xcodeproj -scheme KaChat -destination 'platform=iOS Simulator,name=iPhone 17' test

# Clean build
xcodebuild -project KaChat.xcodeproj -scheme KaChat clean
```

## Architecture

### MVVM Pattern with Singleton Services

The app uses MVVM architecture with global singleton services injected via SwiftUI's `@EnvironmentObject`:

- **Entry Point**: `KaChatApp.swift` initializes all services as `@StateObject` and injects them into the view hierarchy
- **Services**: Singleton pattern (`*.shared`) for core business logic, all marked `@MainActor` for thread safety
- **ViewModels**: `@ObservableObject` classes that bridge services and views
- **Views**: Pure SwiftUI views organized by feature (Chat, Contacts, Onboarding, Settings)

### Key Services

| Service | Purpose |
|---------|---------|
| `WalletManager` | Wallet lifecycle, key derivation, balance tracking |
| `ChatService` | Message polling, conversation state, sync with indexer |
| `NodePoolService` | Main entry point for gRPC node pool (POOLS_v2 architecture) |
| `NodeRegistry` | Persistent node storage with health/profile tracking |
| `NodeSelector` | Capability-aware node selection with scoring |
| `NodeProfiler` | Node discovery, DNS resolution, health probing |
| `GRPCConnectionPool` | Manages gRPC connections with circuit breakers |
| `UtxoSubscriptionManager` | UTXO subscriptions with failover and keepalive |
| `KasiaAPIClient` | REST HTTP client for Kasia Indexer |
| `KeychainService` | Device-specific Secure Enclave credential storage |
| `MessageStore` | Core Data + CloudKit message persistence with per-wallet zones |
| `ContactsManager` | Address book persistence, KNS domain integration |
| `KNSService` | Kaspa Name Service API client for domain resolution |
| `KasiaTransactionBuilder` | Constructs signed Kaspa transactions |

### Messaging Protocol

1. **Handshake**: Initial key exchange, stored in sender's self-stash on-chain
2. **Contextual Messages**: Encrypted messages using shared secret derived from handshake
3. **Payments**: On-chain KAS transfers with optional encrypted metadata
4. **Audio**: Voice messages encoded with YbridOpus codec

### Network Communication

- **Kaspa Node (gRPC)**: Managed by `NodePoolService` (`NodePool/*`) for UTXO subscriptions, transaction submission, and peer discovery
- **Kaspa REST API**: Configurable via Settings (default: `api.kaspa.org` mainnet / `api-tn11.kaspa.org` testnet) for fetching transaction history, payments, and UTXO fallback
- **Kasia Indexer (REST)**: Configurable via Settings (default: `indexer.kasia.fyi`) for message indexing and retrieval
- **KNS API**: Configurable via Settings (default: `api.knsdomains.org`) for Kaspa Name Service domain resolution

### Connection Settings

All network endpoints are configurable in Settings > Connection Settings:
- **Network**: mainnet / testnet toggle
- **Indexer URL**: Kasia message indexer endpoint
- **KNS URL**: Kaspa Name Service API endpoint
- **Kaspa REST API URL**: Block explorer API for transaction lookups

Settings are stored in `AppSettings` struct with network-specific defaults and loaded via `AppSettings.load()` static method (safe to call from any context).

### KNS (Kaspa Name Service) Integration

The app integrates with KNS to provide human-readable domain names for contacts:

**Features:**
- Resolve KNS domains (e.g., `alice.kas`) to Kaspa addresses when adding contacts
- Display KNS domains on contact cards in chat list
- Show all domains owned by a contact in Chat Info view
- Auto-set contact alias to primary KNS domain when not manually set

**API Endpoints:**
- `GET /api/v1/{domain}/owner` - Resolve domain to owner address (forward lookup)
- `GET /api/v1/primary-name/{address}` - Get primary domain for address (reverse lookup)
- `GET /api/v1/assets?owner={address}&type=domain` - Get all domains owned by address

**Key Components:**
- `KNSService` - API client with caching for domain lookups
- `KNSAddressInfo` - Cached info about domains for an address
- `KNSDomainResolution` - Result of forward domain resolution
- `ContactsManager.fetchKNSDomainsForAllContacts()` - Batch fetch for all contacts

### Data Sync Strategy

The app uses a subscription-based approach to minimize polling:

1. **Initial Sync**: Full fetch of historical data on startup/wallet import
2. **gRPC Subscriptions**: Subscribe to `utxosChanged` for real-time payment notifications
3. **Fallback Polling**: If subscription fails, fall back to periodic polling until reconnected
4. **Adaptive per-object cursors** for handshakes/contextual messages:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vsmirn0v/KaChat](https://github.com/vsmirn0v/KaChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
