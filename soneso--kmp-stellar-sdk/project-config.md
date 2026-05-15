---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Kotlin Multiplatform (KMP) project for building a Stellar SDK. The SDK provides APIs to:
- Build Stellar transactions
- Connect to Horizon (Stellar's API server)
- Connect to Stellar RPC Server
- Build smart account wallets with passkey authentication using [OpenZeppelin smart contracts](https://github.com/OpenZeppelin/stellar-contracts)

## Current State

The SDK is **production-ready** with comprehensive functionality implemented:

### Platform Support
- **JVM**: Android API 24+, Server applications (Java 17+)
- **iOS**: iOS 14.0+ (iosX64, iosArm64, iosSimulatorArm64)
- **macOS**: macOS 11.0+ (macosX64, macosArm64)
- **JavaScript**: Browser (WebAssembly) and Node.js 14+

### Core SDK Features (Implemented)
- **Cryptography**: Ed25519 keypair generation, signing, verification
- **StrKey Encoding**: G... (accounts), S... (seeds), M... (muxed), C... (contracts)
- **Transaction Building**: All 27 Stellar operations, TransactionBuilder, FeeBumpTransaction
- **Assets**: Native (XLM), AlphaNum4, AlphaNum12, SAC contract ID derivation
- **Accounts**: Account management, muxed accounts, sequence numbers
- **Horizon Client**: Full REST API coverage, SSE streaming, request builders
- **Soroban RPC**: Contract calls, simulation, state restoration, polling
- **High-Level API**: ContractClient, AssembledTransaction with full lifecycle
- **XDR**: Complete XDR type system and serialization
- **SEP Support**: SEP-1 (Stellar TOML), SEP-2 (Federation Protocol), SEP-5 (Key Derivation), SEP-6 (Deposit and Withdrawal API), SEP-8 (Regulated Assets), SEP-9/12 (KYC), SEP-10 (Web Authentication), SEP-24 (Hosted Deposit/Withdrawal), SEP-30 (Account Recovery), SEP-38 (Anchor RFQ), SEP-45 (Web Authentication for Contract Accounts), SEP-46 (Contract Meta), SEP-47 (Contract Interface Discovery), SEP-48 (Contract Interface Specification), SEP-53 (Sign and Verify Messages)

### Smart Accounts (OpenZeppelin)
- **Contracts**: [OpenZeppelin stellar-contracts](https://github.com/OpenZeppelin/stellar-contracts) v0.7.0
- **SDK Layer**: `stellar-sdk/src/commonMain/.../smartaccount/` with two sub-packages:
  - `core/` — Platform-independent types, builders, errors, signatures, auth payload, CBOR parser
  - `oz/` — OZ-specific kit, managers, clients, config, storage, WebAuthn provider interface
- **Platform Adapters**: WebAuthn providers and storage adapters in `androidMain/`, `jsMain/`, `nativeMain/`
- **Features**: Wallet lifecycle (create, connect, disconnect), multi-signer authorization, context rules, policies (threshold, weighted threshold, spending limit), token transfers, relayer fee sponsoring, indexer credential lookup
- **Documentation**: `docs/smart-accounts/` (API reference, onboarding guide, platform WebAuthn guides)

### Demo Application
- **Platforms**: Android, iOS, macOS, Desktop (JVM), Web
- **Architecture**: Compose Multiplatform with 95% code sharing
- **UI**: Modern celestial-themed design with color-coded cards (purple/gold/teal/blue/red)
- **Features**: 11 comprehensive demos (key generation, funding, account details, trustlines, payments, fetch transaction, contract details, deploy contract, invoke hello world, invoke auth, invoke token contract)
- **Location**: `demo/` directory with platform-specific apps

### Smart Account Demo Application
- **Platforms**: Android, iOS, macOS, Web
- **Architecture**: Compose Multiplatform (Android, iOS, Web) + native SwiftUI (macOS) with Kotlin bridge
- **Features**: Wallet creation/connection, token transfers, context rule management, policy configuration, multi-signer flows
- **Location**: `smart-account-demo/` directory

### Agent Skill
- An [Agent Skill](https://agentskills.io) that teaches AI coding agents how to use this SDK
- **Location**: `skills/kmp-stellar-sdk/` (`SKILL.md` + `references/`)
- **Plugin config**: `skills/.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`

## Architecture Notes

### Cryptographic Implementation

The SDK uses **production-ready, audited cryptographic libraries** - no custom/experimental crypto:

#### JVM Platform
- **Library**: BouncyCastle (`org.bouncycastle:bcprov-jdk18on:1.78`)
- **Algorithm**: Ed25519 (RFC 8032 compliant)
- **Implementation**: `Ed25519Signer`, `Ed25519PrivateKeyParameters`, `Ed25519PublicKeyParameters`
- **Security**: Mature, widely-audited, constant-time operations
- **Provider**: Registered as JCA security provider on initialization

#### Native Platforms (iOS/macOS)
- **Library**: libsodium (via C interop)
- **Algorithm**: Ed25519 (`crypto_sign_*` functions)
- **Implementation**: `crypto_sign_seed_keypair`, `crypto_sign_detached`, `crypto_sign_verify_detached`
- **Security**: Audited, constant-time, memory-safe operations
- **Random**: `randombytes_buf()` using system CSPRNG (`arc4random_buf()`)
- **Distribution**:
  - Framework build: Uses static libsodium from `native-libs/libsodium-ios/`
  - User apps: Add libsodium via Swift Package Manager (Clibsodium)
  - No Homebrew installation required for iOS apps

#### JavaScript Platforms (Browser & Node.js)
- **Library**: libsodium-wrappers-sumo (0.7.13 via npm)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Soneso/kmp-stellar-sdk](https://github.com/Soneso/kmp-stellar-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
