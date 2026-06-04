---
trigger: always_on
description: The EigenLayer Sidecar is an open source, permissionless, verified indexer enabling anyone (AVS, operator, etc) to access EigenLayer's protocol rewards in real-time. It's a sophisticated Go-based application that indexes Ethereum blockchain data, calculates rewards, and provides comprehensive APIs for accessing EigenLayer protocol state.
---

# EigenLayer Sidecar - Claude.ai Development Guide

## Project Overview

The EigenLayer Sidecar is an open source, permissionless, verified indexer enabling anyone (AVS, operator, etc) to access EigenLayer's protocol rewards in real-time. It's a sophisticated Go-based application that indexes Ethereum blockchain data, calculates rewards, and provides comprehensive APIs for accessing EigenLayer protocol state.

**Key Capabilities:**
- Real-time blockchain indexing and state management
- Complex rewards calculation engine
- RESTful and gRPC API services
- Database snapshot creation and restoration
- Multi-network support (mainnet, holesky, testnet, preprod)

## Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Ethereum      │────│   Sidecar        │────│   PostgreSQL    │
│   RPC Node      │    │   Indexer        │    │   Database      │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                              │
                              │
                    ┌─────────────────────┐
                    │   API Services      │
                    │   (gRPC + HTTP)     │
                    │   Port 7100/7101    │
                    └─────────────────────┘
```

### Core Components

- **Indexer**: Fetches and processes blockchain data from Ethereum nodes
- **State Manager**: Manages EigenLayer protocol state (operators, stakers, rewards)
- **Rewards Engine**: Complex calculation system for reward distribution
- **API Server**: gRPC and HTTP endpoints for data access
- **Database**: PostgreSQL with extensive migration system
- **Snapshot System**: Database backup/restore functionality

## Project Structure

```
/
├── cmd/                    # CLI commands and main entry points
├── pkg/                    # Core application packages
│   ├── eigenState/         # Protocol state management
│   ├── rewards/            # Rewards calculation engine
│   ├── rpcServer/          # API server implementation
│   ├── indexer/            # Blockchain indexing logic
│   ├── postgres/           # Database layer and migrations
│   ├── clients/            # External service clients (Ethereum, Etherscan)
│   └── sidecar/            # Main application orchestration
├── docs/                   # Docusaurus documentation site
├── examples/               # Usage examples and client code
├── scripts/                # Development and deployment scripts
├── snapshots/              # Database snapshot configurations
├── charts/                 # Kubernetes Helm charts
└── internal/               # Internal utilities and configuration
```

## Technology Stack

- **Language**: Go 1.23.6
- **Database**: PostgreSQL with GORM ORM
- **APIs**: gRPC with Protocol Buffers, HTTP/REST
- **Blockchain**: Ethereum RPC integration
- **Documentation**: Docusaurus with OpenAPI specs
- **Containerization**: Docker and Docker Compose
- **Deployment**: Kubernetes with Helm charts

## Development Setup

### Prerequisites
- Go 1.23.6+
- PostgreSQL 16+
- Docker and Docker Compose
- Node.js 18+ (for documentation)

### Quick Start
```bash
# Install dependencies
make deps

# Start PostgreSQL with Docker
docker-compose up postgres -d

# Build the application
make build

# Run database migrations
./bin/sidecar database

# Start the sidecar (requires Ethereum RPC URL)
./bin/sidecar run \
  --ethereum.rpc-url=<YOUR_ETH_RPC_URL> \
  --chain=mainnet
```

### Environment Variables
Key configuration via environment variables (prefix: `SIDECAR_`):

```bash
SIDECAR_ETHEREUM_RPC_URL=http://localhost:8545
SIDECAR_CHAIN=mainnet                    # mainnet, holesky, testnet, preprod
SIDECAR_DATABASE_HOST=localhost
SIDECAR_DATABASE_PORT=5432
SIDECAR_DATABASE_USER=sidecar
SIDECAR_DATABASE_PASSWORD=sidecar
SIDECAR_DATABASE_DB_NAME=sidecar
SIDECAR_DEBUG=false
```

## Key Commands

### Building and Testing
```bash
make build                    # Build the sidecar binary
make test                     # Run all tests
make test-file FILE=path.go   # Run specific test file
make lint                     # Run linter
make fmt                      # Format code
make fmtcheck                 # Check code formatting
```

### Database Operations
```bash
./bin/sidecar database           # Run migrations
./bin/sidecar createSnapshot     # Create database snapshot
./bin/sidecar restoreSnapshot    # Restore from snapshot
```

### Running Services
```bash
./bin/sidecar run               # Full indexer + API server
./bin/sidecar rpc               # API server only
```

### Development Tools
```bash
make docs/dev                   # Start documentation site
make docker-buildx-self         # Build Docker image locally
```

## API Endpoints

### gRPC Services (Port 7100)
- **Events**: Block and state change streaming
- **Rewards**: Reward calculation and distribution data
- **Protocol**: General protocol state queries
- **Health**: Service health checks

### HTTP API (Port 7101)
RESTful endpoints with OpenAPI documentation available at `/docs`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Layr-Labs/sidecar](https://github.com/Layr-Labs/sidecar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
