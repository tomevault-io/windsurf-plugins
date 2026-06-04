---
trigger: always_on
description: 1. **NO MOCKING**: Never implement mock data, fake signatures, or placeholder functionality. Everything must be real and production-ready.
---

# Planet Nine Ecosystem - Claude Context

## ⚡ COMMANDMENTS ⚡

1. **NO MOCKING**: Never implement mock data, fake signatures, or placeholder functionality. Everything must be real and production-ready.
2. **NO FALLBACKS**: When something fails, it fails clearly with informative error messages. No fallback data that masks real issues.
3. **NO GIT COMMANDS**: Never run git commands (add, commit, push, etc.) unless explicitly requested by the user. Always ask permission first.

## Overview

The Planet Nine ecosystem is a comprehensive collection of open-source software designed to create an interoperable, privacy-focused alternative to traditional web services. The ecosystem consists of three main layers: **allyabase** (backend services), **The Stack** (core protocols), and **the-nullary** (client applications).

## Core Philosophy

- **Interoperability over Federation**: Unlike federated systems (like ActivityPub), Planet Nine aims for true interoperability where users can connect across services without being locked into specific platforms
- **Privacy by Design**: Uses cryptographic keys for authentication without requiring personal information like emails or passwords
- **Open Source & Self-Hostable**: All components can be self-hosted, avoiding vendor lock-in
- **No Advertising Model**: Designed to work without surveillance capitalism or advertising revenue
- **Production Code Only**: No mock data, test stubs, or demo content in production applications - all implementations must use real service integrations

## 📚 Detailed Documentation

This main document provides an overview. For detailed information, see our modular documentation:

- **[Allyabase Services](docs/ALLYABASE-SERVICES.md)** - Complete backend service documentation (12+ microservices)
- **[The Stack Protocols](docs/THE-STACK-PROTOCOLS.md)** - sessionless, MAGIC, teleportation, covenant protocols
- **[The Nullary Apps](docs/THE-NULLARY-APPS.md)** - All client applications and SVG component system
- **[Development Environment](docs/DEVELOPMENT-ENVIRONMENT.md)** - Setup, testing, and deployment
- **[Testing Ecosystem](docs/TESTING-ECOSYSTEM.md)** - Complete 6-phase testing system

## Architecture Overview

### 1. **allyabase** (Backend as a Service)
**Location**: `/allyabase/`

12+ microservices providing backend functionality:
- **BDO**: Big Dumb Object storage - cryptographically signed distributed storage
- **Sanora**: Product hosting with marketplace and teleportation endpoints
- **Addie**: Payment processing with multi-party transaction splitting
- **Fount**: MAGIC protocol integration and nineum currency management
- **Covenant**: Multi-party contract management with automatic SVG generation
- **Dolores**: Social feeds and PostWidget component system
- **Plus**: Joan (recovery), Julia (P2P messaging), Minnie (email), Pref (preferences), Aretha (limited products), Continuebee (state verification)

### 2. **The Stack** (Core Protocols)
Foundational protocols enabling interoperability:

#### **sessionless** - Passwordless authentication using secp256k1 keys
#### **MAGIC** - Multi-device consensus protocol for secure transactions
#### **teleportation** - Content discovery and verification across bases
#### **covenant** - Multi-party contract management with cryptographic security

### 3. **the-nullary** (Client Applications)
**Location**: `/the-nullary/`

16+ cross-platform applications built with Tauri and SVG-first architecture:
- **Rhapsold**: Minimalist blogging platform (flagship reference implementation)
- **Ninefy**: Digital goods marketplace with type-specific product forms
- **StackChat**: P2P messaging with cross-base communication
- **MyBase**: Personal base management and content aggregation
- **Nexus**: Web-based ecosystem portal showcasing all Planet Nine services
- **Plus**: Covenant, Screenary, IDO This, Grocary, Viewary, Lexary, Photary, Blogary, Viewaris, Eventary, MagiCard

### 4. **the-advancement** (Browser Extensions)
**Location**: `/the-advancement/`

Privacy-focused browser extensions serving as the consumer gateway:
- **Safari Extension**: Complete Planet Nine integration with native cryptography
- **Chrome Extension**: Input detection and typing simulation
- **Payment Processing**: Multi-party Stripe integration (70% creator, 20% base, 10% site)
- **Ad Covering System**: Dual-mode experience (peaceful plants OR interactive monsters)
- **MAGIC Protocol**: Universal spell casting across applications
- **Emojicoding**: Revolutionary UUID ↔ emoji conversion system

## Quick Start

### Running Services Locally
```bash
# Docker (recommended)
cd allyabase/deployment/docker
./test-complete-ecosystem.sh

# Individual services
cd allyabase/[service-name]
npm install && node src/server/node/[service].js

# PM2 for all services
pm2 start ecosystem.config.js
```

### Building Client Apps
```bash
# Nullary applications
cd the-nullary/[app-name]
npm run tauri dev

# Browser extensions
cd the-advancement/src/extensions/safari
swift build -c release
```

### Testing
```bash
# Complete ecosystem testing (6-phase validation)
cd allyabase/deployment/docker
./test-complete-ecosystem.sh

# Individual service tests
cd allyabase/[service]/test/mocha

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [freyja-love-and-magic/freyja-love-and-magic](https://github.com/freyja-love-and-magic/freyja-love-and-magic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
