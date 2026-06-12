---
trigger: always_on
description: HermesVault is a privacy-focused frontend for conducting private transactions on the Algorand blockchain. The application combines a Go backend web server with JavaScript frontend components, utilizing zero-knowledge proofs for transaction privacy.
---

## Project Overview

HermesVault is a privacy-focused frontend for conducting private transactions on the Algorand blockchain. The application combines a Go backend web server with JavaScript frontend components, utilizing zero-knowledge proofs for transaction privacy.

The system consists of two processes:
1. **Go webserver**: Serves frontend and manages backend (creating zk-proofs, sending blockchain transactions)
2. **Python subscriber service**: Monitors blockchain through algod node and saves transaction data

Production also uses systemd path/service units for operational alerting, but those are not part of the application request path.

## Architecture

This is a monorepo containing both backend (Go) and frontend (JavaScript) components:

### Backend Structure
- **Main Application**: `/main.go` - HTTP server with graceful shutdown
- **Handlers**: `/handlers/` - HTTP request handlers for deposit, withdrawal, confirmation, and stats
- **Database Layer**: `/db/` - SQLite database operations with encryption support
- **AVM Integration**: `/avm/` - Algorand Virtual Machine integration and transaction handling
- **ZKP System**: `/zkp/` - Zero-knowledge proof circuits for deposits and withdrawals
- **Models**: `/models/` - Data models for addresses, amounts, notes, and application state
- **Configuration**: `/config/` - Application configuration and smart contract definitions
- **Memory Store**: `/memstore/` - In-memory store for user session data
- **Subscriber Service**: `/subscriber-service/` - Python service to monitor blockchain and update database

### Frontend Structure
- **JavaScript Source**: `/frontend/js/` - Wallet integration, behaviors, and HTMX entry points
- **Templates**: `/frontend/templates/` - HTML templates for deposit, withdrawal, confirmation screens
- **Static Assets**: `/frontend/static/` - Bundled JavaScript, CSS, and images

### Key Technologies
- **Backend**: Go with net/http (no third-party frameworks), SQLite databases, gnark for zero-knowledge proofs, Algorand SDK
- **Frontend**: HTMX for dynamic interactions, esbuild for bundling, Algorand wallet integrations (Pera, Defly, Lute)
- **Privacy**: Zero-knowledge proofs for transaction privacy, encrypted database storage
- **External Services**: nodely.io for algod node connection
- **External JS Modules**: `@pera/connect` for wallet connections, `algosdk` for Algorand transactions, `htmx` for server-driven interactions, `htmx-ext-response-targets` for response target management

### Database Architecture

The system uses two SQLite databases:

**txns.db** (written by Python subscriber service, read by Go webserver):
- `txns` table: Transaction data with leaf_index, commitment, txn_id, txn_type, address, amount, from_nullifier
- `noninserted_withdrawals` table: `no_change=true` withdrawals that do not add Merkle leaves but still affect withdrawal and fee totals
- `stats` table: Global statistics (total deposits, withdrawals, fees)
- `watermark` table: Block sync watermark with algod
- `roots` table: Last merkle tree root and leaf count

**internal.db** (accessed only by Go webserver):
- `notes` table: Note data with leaf_index, commitment, encrypted nullifier, txn_id
- `unconfirmed_notes` table: Temporary storage for unconfirmed transactions

Purpose: txns.db provides merkle proof data for withdrawals; internal.db stores encrypted nullifiers for compliance. Nullifiers are encrypted with a public key stored on disk, while the private key is kept off the server to protect disclosure even if the host is compromised.

## Development Commands

### Frontend Build Commands
```bash
# Build all frontend assets
cd frontend && npm run build

# Build individual components
cd frontend && npm run build:wallet    # Wallet integration bundle
cd frontend && npm run build:behaviors # UI behaviors bundle
cd frontend && npm run build:htmx     # HTMX entry point bundle
cd frontend && npm run copy:missingcss # Copy CSS framework
```

### Go Application
```bash
# Development - run with hot reload using air
air

# Production - run directly
go run main.go

# Run tests
go test ./...
```

### Python Subscriber Service
```bash
# Development - from subscriber-service directory
pipenv run python main.py
```

### Deployment
```bash
# Production deployment
./redeploy.sh
```

### Production Operations

- Apache2 is the public reverse proxy; the Go webserver listens locally on port 5555.
- Nodely whitelisting depends on outbound Algorand API requests carrying `Referer: https://hermesvault.org`. Keep `AlgorandApiReferer` set in `config/.env` for Go and Python algod/indexer clients.
- Apache serves `/var/www/hermesvault/maintenance.html` with status 503 when `/var/www/hermesvault/maintenance.enable` exists.
- Apache also enters maintenance mode when `/home/gws/dev/algorand/hermes/HermesVault-frontend/data/txns/subscriber.fatal` exists.
- The Go webserver has a `subscriberHealthGuard` fallback that returns 503 when `subscriber.fatal` exists, mainly for direct localhost access or Apache bypasses.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giuliop/HermesVault-frontend](https://github.com/giuliop/HermesVault-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
