---
trigger: always_on
description: This document provides guidance for AI agents working on the Reown .NET repository.
---

# AI Agent Guidance for Reown .NET

This document provides guidance for AI agents working on the Reown .NET repository.

## Project Overview

Reown .NET is a monorepo containing Unity and NuGet packages for integrating Web3 functionality into applications. The SDK enables developers to connect to blockchain wallets, execute transactions, sign messages, and interact with smart contracts across multiple EVM and Solana blockchain networks.

The repository contains two main product lines:

**AppKit for Unity** - A comprehensive Unity SDK that provides wallet connection UI, blockchain interactions, and cross-platform support (iOS, Android, Windows, macOS, WebGL). It supports 300+ cryptocurrency wallets via the WalletConnect protocol, social logins (Google, X/Twitter, Discord, Apple, GitHub), and both EVM chains (Ethereum, Optimism, Arbitrum, Polygon, Avalanche, Base, Ronin) and Solana.

**WalletKit** - A pure .NET library for building wallet applications that can receive and respond to WalletConnect requests. Published to NuGet for use in non-Unity .NET applications.

The SDK employs a dual-platform strategy: native platforms (iOS, Android, desktop) use Nethereum for full .NET blockchain capabilities, while WebGL builds use a JavaScript bridge to Wagmi/Viem libraries.

### Combined NuGet + UPM Distribution

This repository uses a combined distribution approach:

**NuGet Packages** - Core .NET packages (`Reown.Sign`, `Reown.WalletKit`, `Reown.Core.*`) are published to nuget.org for use in standard .NET applications. These are built from the `Reown.NoUnity.slnf` solution filter.

**Unity Package Manager (UPM)** - Unity packages (`com.reown.appkit.unity`, `com.reown.sign.unity`, etc.) are distributed via OpenUPM and can be installed using the OpenUPM CLI or Unity Package Manager. Git tags in the format `package-name/version` enable direct UPM installation from the repository.

Both distribution channels share the same source code in the `src/` directory. Unity packages include additional Unity-specific code (MonoBehaviours, ScriptableObjects, Editor scripts) alongside the core .NET logic.

## Repository Structure

```
reown-dotnet/
├── src/                              # Source code for all packages
│   ├── Reown.AppKit.Unity/           # Main Unity SDK (highest-level API)
│   ├── Reown.AppKit.Solana.Unity/    # Solana support for Unity
│   ├── Reown.Sign/                   # WalletConnect protocol core
│   ├── Reown.Sign.Unity/             # Unity-specific signing
│   ├── Reown.Sign.Nethereum/         # Ethereum integration for .NET
│   ├── Reown.Sign.Nethereum.Unity/   # Ethereum integration for Unity
│   ├── Reown.Core/                   # Core framework aggregator
│   ├── Reown.Core.Common/            # Shared utilities and logging
│   ├── Reown.Core.Crypto/            # Encryption and key management
│   ├── Reown.Core.Network/           # HTTP client and JSON-RPC
│   ├── Reown.Core.Network.WebSocket/ # WebSocket communication (NuGet-only, no UPM package)
│   ├── Reown.Core.Storage/           # Data persistence abstractions
│   ├── Reown.WalletKit/              # Wallet SDK for .NET
│   ├── Reown.Unity.Dependencies/     # External dependency aggregator for Unity
│   └── Directory.Build.props         # Central version and build config
│
├── sample/                           # Main sample application
│   └── Reown.AppKit.Unity/           # Primary sample app used for testing and demos
│
├── playground/                       # Additional Unity sample projects (10 total)
│   ├── Reown.AbstractSample.Unity/   # Standard AppKit demo
│   ├── Reown.Customization.Unity/    # Custom UI/branding demo
│   ├── Reown.Playground.Unity/       # General experimentation sandbox
│   ├── Reown.RoninSample.Unity/      # Ronin blockchain demo
│   ├── Reown.SeiSample.Unity/        # Sei blockchain demo
│   ├── Reown.SmartSession.Unity/     # Smart account/session demo
│   ├── Reown.SolanaCore.Unity/       # Solana core integration demo
│   ├── Reown.SolanaSdk.Unity/        # Solana Unity SDK adapter demo
│   ├── Reown.UniTask.Unity/          # UniTask/async patterns demo
│   └── Reown.ZKCandySample.Unity/    # Zero-knowledge proofs demo
│
├── test/                             # Test projects (target net8.0;net9.0;net10.0)
│   ├── Reown.Core.Common.Test/
│   ├── Reown.Core.Crypto.Test/
│   ├── Reown.Core.Network.Test/
│   ├── Reown.Core.Storage.Test/
│   ├── Reown.Sign.Test/
│   ├── Reown.WalletKit.Test/
│   └── Rown.TestUtils/               # Shared test utilities (note: typo in actual directory name)
│
├── .github/
│   ├── workflows/                    # CI/CD automation
│   │   ├── unity-build-test.yml      # Unity builds (Windows/Android/WebGL) + Vercel deploy
│   │   ├── dotnet-build-test.yml     # .NET unit and integration tests
│   │   ├── release.yml               # NuGet + Git tag releases (on merge to main)
│   │   ├── sync-unity-package-version.yml  # Version propagation
│   │   ├── sonarcloud.yml            # Code quality analysis
│   │   ├── claude-review.yml         # AI code review on PRs
│   │   └── cta.yml                   # CTA assistant automation
│   ├── actions/                      # Reusable GitHub Actions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reown-com/reown-dotnet](https://github.com/reown-com/reown-dotnet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
