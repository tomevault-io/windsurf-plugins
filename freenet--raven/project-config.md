---
trigger: always_on
description: Generates/stores an ML-DSA-65 keypair via Freenet's encrypted secret storage.
---

# Freenet Microblogging – Agent Guide

## Overview

Decentralized Twitter/X-like microblogging application built on Freenet. Uses a
TypeScript web UI with Vite, parameterized Rust WASM shard contracts (per-owner /
per-thread) for posts/profile/follows/likes/notifications, an ML-DSA-65 identity
delegate for signing, and the `@freenetorg/freenet-stdlib`
TypeScript SDK for WebSocket communication with a Freenet node.

## Quick Reference

### Commands

```bash
# Build
cargo make build                # Full build: contracts + UI + web container
cargo make build-contracts      # user/thread/inbox shards + identity (WASM + code hashes)
cargo make build-ui             # Vite/TypeScript build (depends on build-contracts)
cargo make build-web-container  # web/container Rust → WASM
cargo make build-ui-offline     # Vite build with mock data (no Freenet node) — for CI

# Publish (local node). Shards are PARAMETERIZED (per owner / per thread) and are
# instantiated on demand by the web app, so there is nothing to publish globally —
# only the identity delegate + the webapp are published.
cargo make publish-identity     # Publish identity delegate
cargo make publish-webapp-test  # Publish test-signed webapp from published-contract/
cargo make publish-all          # End-to-end: build → sign-test → snapshot → publish delegate + webapp

# Publish (PRODUCTION — use scripts/release.sh, not directly)
cargo make publish-production   # Build → sign with prod key → snapshot → publish to live network

# Release
scripts/release.sh 0.1.0        # End-to-end release driver (preflight + 3 confirmation gates)

# Development
cd web && npm run dev           # Vite dev server on :8080
cargo make dev-offline          # Vite dev server with mock data (no node required)

# Quality
cargo make test                 # Rust tests + Vitest
cargo make clippy               # Workspace clippy, deny warnings
cargo make fmt-check            # cargo fmt --check
cargo make check                # cargo check + tsc --noEmit

# Playwright
cargo make test-ui-playwright-setup  # One-time browser install
cargo make test-ui-playwright        # Run E2E suite

# Node
cargo make run-node             # Local Freenet node
```

### Repository Structure

```
freenet-microblogging/
├── contracts/                  # Parameterized shard contracts (ADR-0001)
│   ├── user-shard/             # Per-owner: posts, profile, follows (owner-writes)
│   ├── thread-shard/           # Per-root-post: replies, likes, quotes (anyone-writes)
│   └── inbox-shard/            # Per-owner: notifications (anyone-writes, owner-prunes)
├── delegates/
│   └── identity/               # Identity delegate (Rust → WASM)
│       ├── src/lib.rs          # ML-DSA-65 keypair, post/like signing
│       ├── Cargo.toml
│       └── freenet.toml
├── web/                        # TypeScript web frontend
│   ├── index.html              # App entry point (Vite serves this)
│   ├── vite.config.ts          # Vite bundler config
│   ├── src/
│   │   ├── index.ts            # Entry: mounts app shell
│   │   ├── app.ts              # App shell: assembles 3-column layout
│   │   ├── types.ts            # Post, User, TrendingTopic interfaces
│   │   ├── mock-data.ts        # Mock posts/users for development
│   │   ├── theme.ts            # Dark/light mode toggle
│   │   ├── utils.ts            # formatRelativeTime helper
│   │   ├── vite-env.d.ts       # Vite type declarations
│   │   ├── components/
│   │   │   ├── sidebar.ts      # Logo, nav, theme toggle, post CTA
│   │   │   ├── feed.ts         # Tab bar, compose, post list, filtering
│   │   │   ├── compose-box.ts  # Textarea, char counter, post button
│   │   │   ├── post-card.ts    # Post card with actions, timestamps
│   │   │   ├── right-panel.ts  # Search, trending, who-to-follow
│   │   │   └── bottom-nav.ts   # Mobile bottom navigation
│   │   └── scss/
│   │       ├── styles.scss     # Main entry (imports all partials)
│   │       ├── _variables.scss # CSS custom properties (design tokens)
│   │       ├── _reset.scss     # Minimal reset
│   │       ├── _layout.scss    # 3-column grid
│   │       ├── _sidebar.scss   # Sidebar styles
│   │       ├── _feed.scss      # Feed, compose, post cards
│   │       ├── _right-panel.scss # Trending, follow cards
│   │       ├── _buttons.scss   # Button variants
│   │       ├── _dark-mode.scss # Dark mode overrides
│   │       └── _responsive.scss # Mobile/tablet breakpoints
│   ├── container/              # Web contract container (Rust → WASM)
│   │   └── src/lib.rs
│   ├── package.json
│   ├── tsconfig.json
│   └── freenet.toml
├── Cargo.toml                  # Workspace root
├── Makefile.toml               # Build orchestration (cargo-make)
├── DESIGN.md                   # Visual design system specification
├── CLAUDE.md                   # → points to this file
└── AGENTS.md                   # This file (single source of truth)
```

### Key Dependencies

| Dependency | Purpose |
|-----------|---------|
| `@freenetorg/freenet-stdlib` | Freenet TypeScript SDK — WebSocket API, FlatBuffers types |
| `vite` | Build tool and dev server |
| `vitest` | Test runner |
| `typescript` | Language |
| `sass` | SCSS compilation |
| `freenet-stdlib` (Rust) | Contract/delegate traits, WASM macros |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [freenet/raven](https://github.com/freenet/raven) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
