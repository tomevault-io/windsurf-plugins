---
trigger: always_on
description: Web3 Icons is a monorepo providing comprehensive cryptocurrency, blockchain, and Web3-related icons as SVGs and React components. The project uses Bun as the package manager and Turbo for build orchestration.
---

# Web3Icons

Web3 Icons is a monorepo providing comprehensive cryptocurrency, blockchain, and Web3-related icons as SVGs and React components. The project uses Bun as the package manager and Turbo for build orchestration.

## Common Commands

### Build Commands

```bash
# Build all packages and apps (runs turbo build + post-build)
bun build

# Build only packages (common, core, react)
bun build:packages

# Build individual packages
bun build:common    # Generates metadata TypeScript files from JSON
bun build:core      # Optimizes SVGs, generates index and svg-module
bun build:react     # Generates React components from SVGs

# Build apps
bun build:apps
bun build:website   # Next.js website
bun build:figma     # Figma plugin

# Post-build step (runs after main build)
bun post-build      # Runs generate-icons-md + format
```

### Development

```bash
# Start development mode (uses turbo dev)
bun dev

# Format code (Prettier)
bun format
```

### Icon Management

```bash
# Add new icons - detects git changes in raw-svgs/
bun add-icons

# Add metadata entry that references existing icons (e.g., testnets, wrapped tokens)
bun add-metadata

# Delete icons and their metadata
bun delete-icons

# Check for missing metadata
bun check-missing

# Validate SVG files
bun validate-svgs
bun validate-filenames
```

### Documentation

```bash
# Generate icons documentation (docs/icons.md)
bun generate-icons-md
```

### Publishing

```bash
# Sync to CDN (R2)
bun cdn:sync
```

## Architecture

### Monorepo Structure

```
web3icons/
├── packages/
│   ├── common/          # Shared metadata and types
│   ├── core/            # Optimized SVGs
│   └── react/           # React components
├── apps/
│   ├── website/         # Next.js website (web3icons.io)
│   └── figma-plugin/    # Figma plugin
├── scripts/             # Build scripts and CLI tools
└── raw-svgs/            # Source SVG files
```

### Package Dependencies

- `@web3icons/common`: Base package containing metadata (tokens.json, networks.json, wallets.json, exchanges.json) and TypeScript types
- `@web3icons/core`: Depends on `common`. Contains optimized SVGs in `src/svgs/`
- `@web3icons/react`: Depends on `common`. Contains generated React components

### Icon Types and Variants

**Types**: `token`, `network`, `wallet`, `exchange`

**Variants**: `branded` (color), `mono` (monochrome), `background` (tokens only)

### Build Process Flow

#### 1. Common Package Build

- Runs `scripts/build-scripts/generate-metadata.ts`
- Converts JSON metadata files to TypeScript files with type annotations
- Located in `packages/common/src/metadata/`

#### 2. Core Package Build

Pre-build steps (`scripts/build-scripts/core/core.pre-build.ts`):

1. **Order metadata**: Alphabetically sorts all metadata JSON files
2. **Optimize SVGs**: Processes raw-svgs/ → packages/core/src/svgs/
   - Applies SVGO optimization
   - Organizes by type and variant folders
3. **Generate svg-module.ts**: Creates import mappings for all SVGs
4. **Generate index.ts**: Exports all SVG assets

#### 3. React Package Build

Pre-build steps (`scripts/build-scripts/react/react.pre-build.ts`):

1. **Generate components**: Converts SVGs to React components using SVGR
   - Creates `Token{Symbol}`, `Network{Name}`, `Wallet{Name}`, `Exchange{Name}` components
   - Located in `packages/react/src/icons/`
2. **Generate index.ts**: Exports all React components

### Icon Workflow

1. **Add SVG files** to `raw-svgs/{type}/{variant}/` following naming conventions:
   - Tokens: UPPERCASE symbol (e.g., `BTC.svg`)
   - Networks/Wallets/Exchanges: kebab-case (e.g., `binance-smart-chain.svg`)
   - SVGs must be 24x24px frames (see CONTRIBUTING.md)

2. **Run `bun add-icons`**:
   - Detects git changes in raw-svgs/
   - Prompts for metadata (name, id, CoinGecko ID, etc.)
   - Adds entries to metadata JSON files
   - Or use `bun add-metadata` to reference existing icons

3. **Build packages**:
   - `bun build:packages` processes everything
   - SVGs are optimized and copied to core package
   - React components are auto-generated

### Metadata Structure

Metadata files are the source of truth stored as JSON in `packages/common/src/metadata/`. They are converted to TypeScript files during the common package build.

#### Token Metadata (`tokens.json`)

```json
{
  "id": "bitcoin",
  "filePath": "token:BTC",
  "symbol": "BTC",
  "name": "Bitcoin",
  "marketCapRank": 1,
  "addresses": {
    "ethereum": "0x2260fac5e5542a773aa44fbcfedf7c193bc2c599",
    "binance-smart-chain": "0x7130d2a12b9bcbfae4f2634d864a1ee1ce3ead9c"
  },
  "variants": ["branded", "mono", "background"]
}
```

**Fields**:

- `id`: Unique identifier (typically lowercase with hyphens)
- `filePath`: Icon reference in format `type:filename` (e.g., `token:BTC`)
  - **Icon Sharing System**: Multiple entries can reference the same SVG file
  - Example: Wrapped tokens can use `filePath: "token:ETH"` to share Ethereum's icon
  - Example: Testnets can use `filePath: "network:ethereum"` to share mainnet's icon
- `symbol`: Token ticker (uppercase, e.g., `BTC`, `ETH`)
- `name`: Full token name

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xa3k5/web3icons](https://github.com/0xa3k5/web3icons) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
