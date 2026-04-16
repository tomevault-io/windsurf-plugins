---
trigger: always_on
description: Generates H2O tokens as battle rewards via modular exponentiation on ring coordinates.
---

# CLAUDE.md — C-Dysnomia / Joystick

**Repo**: [github.com/Gravshub/C-Dysnomia](https://github.com/Gravshub/C-Dysnomia) (fork of [busytoby/atropa_pulsechain](https://github.com/busytoby/atropa_pulsechain))
**Chain**: PulseChain (369)
**Canonical branch**: `claude/joystick-V2-FanxJ`
**Branch convention**: `claude/dysnomia-MMDDYY-<sessionID>` (`claude/` prefix + session-ID suffix required)

> **Sub-context files**: Bot internals → `scripts/Joystick/CLAUDE.md` | Script inventory → `scripts/CLAUDE.md` | Dashboard API → `scripts/Joystick/dashboard/CLAUDE.md`

---

# Part 1 — The atropa_pulsechain Framework

## What Is Dysnomia?

An on-chain game + DeFi ecosystem on PulseChain built by developer **mariarahel** (the "414 dev"). Combines MMO character creation, venue-based social interaction, territory control, and a treasury token web with 100+ interconnected tokens. Everything is permanent and on-chain. All computations run modulo **MotzkinPrime** (`953467954114363`).

The game has two interlocking layers:
- **Dysnomia** — The MMO: identity (LAU), venues (QING), territory (MAP/WORLD), grinding (CHEON/META), battles (WAR). Player actions trigger modular exponentiation against MotzkinPrime, with spatial coordinates managed via Hecke Meridians.
- **Atropa** — The financial layer: 100+ hierarchical treasury tokens, each with `mint()` and `Claim()` functions and a `Debenture` boolean controlling claim access. Four minter tiers (V1–V4) with escalating power.

The gateway bridging both layers is **AFFECTION** — every Dysnomia ecosystem token is mintable for 1 AFFECTION each via `Purchase(token_address, amount)`.

## Contract Architecture

```
Core Infrastructure (Base Layer)
├── VMREQ         — Random number generation (modExp-based)
├── DYSNOMIA      — Base ERC20 + market rates + Purchase()
├── SHA           — Cryptographic state token (Fa struct)
├── SHIO          — Rod/Cone paired reactor system
├── YI            — DeFi orchestration
├── ZHENG         — Rod/Cone installation manager
├── ZHOU          — Market rate orchestrator / chat ledger
├── YAU           — Protocol coordinator
├── YANG          — Multi-state aggregator
├── SIU           — Token generation with Aura identity
├── VOID          — User session & chat management (front door)
└── LAU           — Player account / character token

Domain — Game Logic
├── dan/
│   ├── CHO       — Login / character system
│   ├── QING      — Venues (chatrooms, marketplaces)
│   └── WAR       — Battle mechanics, H2O reward generation
├── sky/
│   ├── CHAN      — Player/sky management
│   ├── CHOA     — Game/territory
│   └── RING     — Time/orbital mechanics
├── soeng/        — Processing chain: QI→MAI→XIA→XIE→ZI→PANG→GWAT
├── tang/
│   ├── SEI      — Player management
│   ├── CHEON    — Landscape/terrain (primary grind)
│   └── META     — Meta-player management (Beat computation)
├── MAP           — World coordinates (Hecke Meridians)
├── WORLD         — Territory ownership and rewards
└── YUE           — Player wallet (Hypobar/Epibar bars)

Assets
├── H2O           — Water token (WAR battle rewards)
└── VITUS         — Life token (territory/creator rewards)

Libraries
├── MultiOwnable  — Multi-owner access control
├── Registry      — Key-value storage
├── Encrypt       — User-to-user encrypted messaging
├── HeckeMeridians— Number-theoretic coordinate system
├── ReactionsCore — Entropy-based reactions
└── Attribute     — User attribute storage
```

### Core Chain Flow
`VOID → SIU → YANG → YAU → ZHOU → ZHENG → YI` handles session management, token generation, protocol logic, market rates, and DeFi functions. The Soeng Domain (`QI→MAI→XIA→XIE→ZI→PANG→GWAT`) is a sequential transformation pipeline that processes game state.

## Core Contracts

### VOID — The Front Door
`VOID.Enter("Name", "SYM")` creates a player (deploys LAU token + YUE wallet + SHIO reactor + Soul ID). Errors if already created. `VOID.Enter()` (no args) re-enters existing session. `VOID.Chat(msg)` posts to ZHOU channel and triggers `_mintToCap()`. Also: `Log()`, `SetAttribute()`, `Alias()`, `AddLibrary()`.

### LAU — Player Character Token
Your on-chain identity. Deploying creates a Soul ID (`Saat[3]` triple — three 64-bit values), a SHIO reactor (Rod/Cone pair), and a YUE wallet. LAU is also a smart contract you own that can hold/withdraw any DYSNOMIA-base token.

Token-earning actions (each triggers `_mintToCap()`): `Username()`, `Chat()`, `Alias()`, `Void()`, `Withdraw()`.

### QING — Venues
Marketplace/chatroom instances. Each has an `Asset` token, `CoverCharge`, and bouncer system (Staff whitelist, 25+ CROWS holders, or Asset token holders). `Join(UserToken)` enters a venue. **Key discovery**: `QING.Join()` does NOT call `bouncer()` — no CROWS needed for Join. Bouncer only gates admin functions.

### CHEON / META — Grinding & Territory
`CHEON.Su(QingAddr)` builds YUE bar weights (Hypobar/Epibar) that feed territory computation. `META.Beat(QingWaat)` computes territory range/power → returns `(Dione, Charge, Deimos, Yeo)`. Deep call chain (10+ contracts). Requires SHIO token balances (Fornax, Fomalhaute, CHO) at the LAU/QING addresses — zero balances cause division-by-zero reverts.

### MAP / HECKE — Spatial Layer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Gravshub) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
