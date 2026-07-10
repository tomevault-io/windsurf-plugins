---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Communitas is a local-first, PQC-ready collaboration platform that merges WhatsApp, Dropbox, Zoom, and Slack into one decentralized application. It uses connection words (four-word networking) to share peer connection details, provides per-entity virtual disks (org, group, channel, project, individual), and enables DNS-free website publishing via identity-bound website roots.

**Platform Focus**: Two applications — a cross-platform Dioxus + Tauri app (macOS, Windows, Linux; experimental Android/iOS) and a native macOS Swift app (`communitas-apple/`). Both connect to a local x0xd daemon for all networking.

## Core Architecture

### Dioxus Application
- **Location**: `communitas-dioxus/`
- **Framework**: Dioxus + Tauri 2 (all-Rust)
- **State Management**: Signals/hooks backed by `communitas-ui-service`
- **Platforms**: macOS, Windows, Linux (GA) with experimental Android/iOS builds via Tauri runners
- **Daemon**: Requires x0xd; onboarding gate auto-installs it on first run

### Swift Application (macOS)
- **Location**: `communitas-apple/`
- **Framework**: SwiftUI, Swift Package Manager, macOS 14+
- **Targets**: `Communitas` (executable) + `X0xClient` (library)
- **Daemon**: Discovers x0xd config from `~/Library/Application Support/x0x/api.port` and `api-token`

### Rust Core Library
- **Location**: `communitas-core/`
- **Purpose**: Cross-platform business logic, P2P networking, cryptography
- **Cryptography**: Post-quantum (ML-DSA/ML-KEM) with ChaCha20-Poly1305
- **Storage**: Virtual disks with CRDT synchronization (Yrs)
- **Networking**: QUIC via ant-quic, IPv4-first with Happy Eyeballs (RFC 8305) dual-stack fallback

### Key Components
- **Connection Words**: Human-readable encoding for sharing IP:port (e.g., "ocean-forest-moon-star")
- **Virtual Disks**: Private/Public/Shared per entity with different encryption policies
- **Website Publishing**: DNS-free web via identity.website_root binding
- **Messaging**: End-to-end encrypted group messaging with editing, deletion, pinning, threading, and inline quotes/replies
- **Emoji Reactions**: Per-message reactions with quick-reaction bar and full categorized emoji picker (with search)
- **Markdown Rendering**: In-message markdown with syntax highlighting
- **@Mentions**: Autocomplete picker with inline user tagging
- **Typing Indicators**: Real-time per-user typing status in channels
- **Presence**: Online/away/offline status badges per peer
- **Message Search**: In-channel search with debounced input and result highlighting
- **Onboarding Gate**: First-run flow that auto-installs and starts x0xd if not present
- **Groups**: Threshold-ready group identities with ML-DSA signatures and member management (add/remove/roles)
- **Kanban System**: CRDT-based collaborative project management (`communitas-kanban/`)
- **Entity Tabs**: Board, Chat, Call, Canvas, Drive, Documents, and Details views per entity type
- **Offline-First**: All operations work locally and sync when network available
- **CRDT Tombstone Compaction**: Configurable retention policies with background compaction tasks
- **Signed Presence Beacons**: ML-DSA signed presence broadcasts with per-peer rate limiting
- **SWIM Failure Detection**: Complete K-peer probing, indirect probes, suspect-to-dead transitions
- **Anti-Entropy Reconciliation**: Set-difference based partition recovery
- **UI Components**: VirtualList, SearchBar, FilterChips, Pagination, ConfirmDialog, ErrorBanner, loading skeletons, empty states

## Development Commands

### Quick Start - Dioxus App
```bash
# Install dx CLI (pinned)
scripts/install_dx.sh

# Run Dioxus desktop app with hot reload
cd communitas-dioxus
dx serve --platform desktop --hotpatch

# Bundle for release
dx bundle --platform desktop
```

### Rust Development
```bash
# Build all Rust crates
cargo build

# Run tests
cargo nextest run

# Format and lint
cargo fmt --all
cargo clippy --all-features -- -D clippy::panic -D clippy::unwrap_used -D clippy::expect_used

# Build specific crates
cargo build -p communitas-core
cargo build -p communitas-kanban
cargo nextest run -p communitas-core
cargo nextest run -p communitas-kanban
```

### Swift App (Native macOS)
```bash
# Build from command line
swift build --package-path communitas-apple

# Open in Xcode
open communitas-apple/Package.swift
```

## Workspace Crates

| Crate | Purpose |
|-------|---------|
| `communitas-core` | Core business logic, P2P, cryptography |
| `communitas-kanban` | CRDT-based Kanban system |
| `communitas-ui-api` | Strongly-typed UI service trait definitions |
| `communitas-ui-service` | Shared Rust UI service implementations (ADR-019) |
| `communitas-x0x-client` | x0xd daemon discovery, HTTP client, WebSocket transport |
| `communitas-bench` | Benchmarks |

## Architecture Insights

### Core Context System
The application uses a centralized `CoreContext` (communitas-core/src/core_context.rs) that wires Communitas to saorsa-gossip v0.5.0 components:
- Identity management with enhanced PQC support
- Storage management with CRDT synchronization (Yrs) and tombstone compaction

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [saorsa-labs/communitas](https://github.com/saorsa-labs/communitas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
