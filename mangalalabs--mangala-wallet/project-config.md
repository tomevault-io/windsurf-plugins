---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mangala Wallet is a Kotlin Multiplatform cryptocurrency wallet supporting Android, iOS, and Desktop platforms. It supports multiple blockchain networks including Antelope (EOS/WAX/Telos), EVM-compatible chains, and Bitcoin.

The project uses three build variants:
- **Pro**: Full-featured wallet (combined functionality)
- **Cold**: Air-gapped signing device for secure transaction signing
- **UI**: Transaction broadcast only (works with Cold variant)

## Essential Commands

### Initial Setup

```bash
# Initialize submodules
git submodule update --init --recursive

# Create local.properties with required API keys (see README.md)
# Required keys: GITHUB_ACTOR, GITHUB_TOKEN, ALCHEMY_API_KEY, COVALENTHQ_API_KEY, INFURA_API_KEY, INFURA_SECRET_KEY
```

### Build Commands

```bash
# Build all modules
./gradlew build

# Run tests
./gradlew test

# Check code style
./gradlew ktlintCheck

# Generate database interfaces after editing .sq files
./gradlew generateCommonMainAntelopeDatabaseInterface
```

### Platform-Specific Builds

**Android**: Use Android Studio build variants

**iOS**:
- Select scheme in Xcode
- If package issues occur: File → Packages → Reset Package Caches
- Pod install: `./gradlew :composeApp:podInstallSyntheticIos`

**Desktop**:
1. Set `desktopBuildType` in `gradle.properties` (debug or release)
2. Run `./gradlew :common:utils:generateBuildKonfig` to update values

### Build Variant Configuration

Set `currentFlavor` in `gradle.properties`:
- `currentFlavor=pro` (default)
- `currentFlavor=cold`
- `currentFlavor=ui`

**IMPORTANT**: Do not commit changes to `currentFlavor` in `gradle.properties`

### Database Modifications

When editing prepopulated database in `./common/mokoresources/src/commonMain/moko-resources/files/`:

```bash
cd ./common/mokoresources/src/commonMain/moko-resources/files/
for file in *.sql; do
    sqlite3 mangalawallet.db < "$file"
done
```

Then create migration in `data/local/src/commonMain/sqldelight/migrations`

## Architecture

### Clean Architecture with MVVM

```
Presentation Layer (Compose UI + ScreenModels)
    ↓
Domain Layer (Use Cases + Domain Models)
    ↓
Data Layer (Repositories + Data Sources)
```

### Module Structure

- `composeApp/` - Main Compose Multiplatform application entry point
- `core/` - Core functionality modules:
  - `core/security/` - Security and cryptography
  - `core/auth/` - Authentication logic
  - `core/hdwallet/` - HD wallet implementation
  - `core/pin/` - PIN management
  - `core/biometry/` - Biometric authentication
  - `core/websocket-chat/` - WebSocket chat functionality
  - `core/ai/` - AI conversation features
- `features/` - Feature modules organized by functionality:
  - `features/wallet_*/` - Wallet management (pro/cold/ui variants)
  - `features/portfolio/` - Portfolio aggregation and display
  - `features/send_*/` - Send transaction flows
  - `features/chains/` - Blockchain-specific implementations
    - `features/chains/evmcompatible/` - EVM chains
    - `features/chains/antelope_*/` - Antelope chains
    - `features/chains/bitcoin/` - Bitcoin support
  - `features/passkey/` - Passkey authentication
  - `features/walletconnect/` - WalletConnect integration
  - `features/dex/` - DEX integrations (Uniswap, etc.)
- `data/` - Data layer:
  - `data/local/` - SQLDelight local database
  - `data/remote/` - Network data sources
  - `data/model/` - Data models
- `domain/` - Domain layer with use cases and domain models
- `libraries/` - Shared libraries (chart, scanqr, walletconnect, etc.)
- `antelope/` - Antelope blockchain SDK modules
- `common/` - Common utilities:
  - `common/ui/` - Shared UI components
  - `common/utils/` - Utility functions
  - `common/mokoresources/` - Resources and assets

### Key Technologies

- **UI**: Jetpack Compose Multiplatform
- **Navigation**: Type-safe navigation with sealed classes
- **DI**: Koin dependency injection
- **Database**: SQLDelight for local storage
- **Serialization**: kotlinx-serialization
- **Networking**: Ktor client
- **State Management**: MVVM with ScreenModels (Cafe Bazaar Voyager)
- **Async**: Kotlin Coroutines and Flow
- **Resources**: MOKO Resources for multiplatform assets

## Git Workflow

**CRITICAL**: Always follow the Git workflow defined in `GIT_WORKFLOW.md` and development standards in `.claude/development-standards.md`.

### Branch Naming

```
<type>/<short-description>
```

Types: `feature/`, `bugfix/`, `hotfix/`, `refactor/`, `docs/`, `test/`, `chore/`, `perf/`

Examples:
- `feature/add-solana-support`
- `bugfix/fix-balance-calculation`
- `hotfix/security-patch`

### Commit Messages

```
<type>(<scope>): <subject>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`, `ci`, `build`, `revert`

Scopes: `auth`, `wallet`, `portfolio`, `transaction`, `network`, `database`, `ui`, `core`, etc.

Examples:
- `feat(portfolio): add multi-wallet aggregation for EVM`
- `fix(transaction): resolve crash when parsing invalid data`
- `refactor(network): extract HTTP client configuration`

### Branching Strategy

- Branch from `develop` for features/bugfixes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MangalaLabs/mangala-wallet](https://github.com/MangalaLabs/mangala-wallet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
