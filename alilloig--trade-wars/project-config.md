---
trigger: always_on
description: Trade Wars is a **learning/educational blockchain game** built on Sui, demonstrating full-stack dApp development with Move smart contracts and a React frontend. This is a **work in progress** project by a **solo developer** intended to be **open sourced**.
---

# CLAUDE.md - Trade Wars Project Guide

## Project Overview

Trade Wars is a **learning/educational blockchain game** built on Sui, demonstrating full-stack dApp development with Move smart contracts and a React frontend. This is a **work in progress** project by a **solo developer** intended to be **open sourced**.

### Game Mechanics
- **Resource Mining**: Three elements (Erbium, Lanthanum, Thorium) are mined from planets over time
- **Mine Upgrades**: Players spend resources to upgrade mines for faster production
- **Universe Expansion**: Players create an Overseer, join universes, and control multiple planets
- **Future Features**: Combat and trading mechanics are planned but not yet implemented

## Project Structure

```
trade-wars/
├── move/                    # Sui Move smart contracts
│   └── sources/
│       ├── overseer.move    # Player character management
│       ├── universe.move    # Game world management
│       ├── planet.move      # Planet and mining logic
│       ├── element_mine.move # Mine production calculations
│       └── *.move           # Token types and utilities
│
├── web/                     # React frontend
│   └── src/
│       ├── components/      # UI components (Tailwind)
│       │   ├── ui/          # Primitives (Card, Button, Badge, Spinner)
│       │   ├── layout/      # Header, Footer, PageContainer
│       │   ├── overseer/    # OverseerList, OverseerEmpire
│       │   └── planet/      # PlanetGrid, PlanetDetailsView
│       ├── contexts/        # React Context (NavigationContext)
│       ├── hooks/           # Custom hooks (useOverseer, usePlanet, etc.)
│       ├── services/sui/    # Transaction builders
│       ├── utils/           # Parsing, formatting, validation
│       ├── types/           # TypeScript type definitions
│       ├── constants/       # Environment, colors, blockchain constants
│       └── pages/           # Page components
│
├── cli/                     # CLI tools and deployment scripts
│   └── .env.testnet         # Testnet deployment IDs
│
├── CLAUDE.md                # This file - project guide for Claude
└── DEPLOYMENTS.md           # Deployment history and object IDs
```

## Development Commands

### Frontend (web/)
```bash
npm run dev      # Start development server (http://localhost:5173)
npm run build    # Build for production (runs tsc + vite build)
npm run lint     # Run ESLint
npm run preview  # Preview production build
```

### Smart Contracts (move/)
```bash
sui move build   # Compile Move contracts
sui move test    # Run Move unit tests
sui client publish --gas-budget 100000000  # Deploy to network
```

## Code Style & Conventions

### TypeScript/React
- **Functional components only** - No class components
- **Strict TypeScript** - Avoid `any` types, use proper typing
- **Tailwind only** - No inline styles or CSS files, only Tailwind classes
- **Minimal comments** - Code should be self-documenting

### File Naming
- Components: PascalCase (`PlanetGrid.tsx`)
- Hooks: camelCase with `use` prefix (`usePlanet.ts`)
- Utils/Services: camelCase (`data-access.ts`)
- Types: camelCase (`game.ts`)

### Import Order
1. React/external libraries
2. Internal components
3. Hooks
4. Utils/services
5. Types/constants

## Sui SDK Best Practices

### Data Access Strategy
| Data Type | Method | Reason |
|-----------|--------|--------|
| Time-dependent (reserves) | `devInspect` + `bcs.U64.parse()` | Computes production based on time |
| Static fields (mine levels) | `getObject` → access fields | No computation needed |
| Vector returns (planet IDs) | `devInspect` + `bcs.vector(bcs.Address).parse()` | Returns computed list |

### Correct SDK Usage
```typescript
// ✅ Use BCS module for parsing
import { bcs } from '@mysten/sui/bcs';
const value = bcs.U64.parse(new Uint8Array(returnValue));
const ids = bcs.vector(bcs.Address).parse(new Uint8Array(returnValue));

// ❌ Don't manually parse bytes
// for (let i = 0; i < 8; i++) { result += bytes[i] * Math.pow(256, i); }
```

### Transaction Building
- Use transaction builders in `services/sui/transactions/`
- Batch multiple calls in one PTB when possible
- Always consider gas costs

## Smart Contract Entry Functions

### Overseer Module
- `vest_overseer()` - Create new player (Overseer)
- `join_universe(overseer, universe, sources, random, clock)` - Join a universe
- `upgrade_erbium_planet_mine(...)` - Upgrade erbium mine
- `upgrade_lanthanum_planet_mine(...)` - Upgrade lanthanum mine
- `upgrade_thorium_planet_mine(...)` - Upgrade thorium mine
- `get_universe_planets(overseer, universe_id)` - Get player's planets in universe

### Planet Module (View Functions)
- `get_*_reserves(planet, clock)` - Get current reserves (time-dependent)
- `get_*_mine_level(planet)` - Get mine level (static)
- `get_*_mine_*_upgrade_cost(planet)` - Get upgrade costs (static)

## Environment Variables

Located in `web/.env`:
```
VITE_TRADE_WARS_PKG_DEV=<package_id>      # Deployed package ID
VITE_TRADE_WARS_INFO_DEV=<info_object_id> # TradeWarsInfo shared object
```

See `DEPLOYMENTS.md` for current testnet IDs.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alilloig) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
