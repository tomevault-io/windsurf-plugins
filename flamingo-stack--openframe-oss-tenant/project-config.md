---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Commands

### Build Commands
```bash
mvn clean install                           # Build all Java services and libraries
mvn clean install -DskipTests              # Build without running tests
mvn test                                    # Run all tests
```

### Frontend (UI) Commands
```bash
cd openframe/services/openframe-frontend
npm install                                 # Install dependencies
npm run dev                                 # Start development server
npm run build                               # Build for production
npm run type-check                          # TypeScript type checking
```

### Rust Client Commands
```bash
cd client
cargo build                                 # Build the Rust client
cargo test                                  # Run Rust tests
cargo run                                   # Run the client locally
```

### Local Development
```bash
# Platform-specific startup scripts:
./scripts/run-mac.sh                        # macOS
./scripts/run-linux.sh                      # Linux
./scripts/run-windows.ps1                   # Windows PowerShell

# Silent mode (no prompts):
./scripts/run-mac.sh --silent
```

### Docker Operations
```bash
# Note: Docker Compose files are located in integrated-tools/ directory
# Individual service stacks can be found in their respective subdirectories:
# - integrated-tools/tactical-rmm/
# - integrated-tools/fleetmdm/
# - integrated-tools/meshcentral/
# - integrated-tools/authentik/
```

## Architecture Overview

OpenFrame is a distributed microservices platform with the following core architecture:

### Service Layer
- **openframe-gateway**: API Gateway with JWT authentication, WebSocket support, and tool proxy
- **openframe-api**: GraphQL API service with OAuth2/OpenID Connect, user management
- **openframe-management**: Administrative service with scheduled tasks and system management
- **openframe-stream**: Stream processing service using Kafka for real-time data processing (NOT NiFi)
- **openframe-config**: Spring Cloud Config Server for centralized configuration management
- **openframe-client** (Java): Agent management and authentication service
- **openframe-frontend**: Next.js 16 + React 19 + TypeScript frontend (consumes the external `@flamingo-stack/openframe-frontend-core` design system)

### Client Agent
- **client/** (Rust): Cross-platform system agent for monitoring and management

### Shared Libraries
- **openframe-data**: Data access layer (MongoDB, Cassandra, Redis, Kafka)
- **openframe-jwt**: JWT security implementation with cookie support
- **api-library**: Common API services and DTOs

### Technology Stack

#### Backend
- **Runtime**: Java 21, Spring Boot 3.3.0, Spring Cloud 2023.0.3
- **API**: GraphQL (Netflix DGS 7.0.0), RESTful services
- **Security**: JWT with OAuth2, Spring Security, AES-256 encryption
- **Data**: MongoDB 7.x, Cassandra 4.x, Apache Pinot 1.2.0, Redis
- **Messaging**: Apache Kafka 3.6.0 for event streaming
- **Processing**: OpenFrame Stream Service (custom data processing components)

#### Frontend
- **Framework**: Next.js 16 + React 19 with TypeScript 5.8 (App Router)
- **UI Library**: `@flamingo-stack/openframe-frontend-core` — external shared design system. No custom UI primitives.
- **Data Fetching**: `@tanstack/react-query` (no Apollo Client)
- **State**: Zustand with immer
- **Forms**: react-hook-form + zod
- **Code Quality**: Biome (primary linter + formatter), Husky pre-commit
- **Build**: Next.js (webpack or Turbopack)
- See `openframe/services/openframe-frontend/CLAUDE.md` for the full conventions.

#### Infrastructure
- **Containerization**: Docker and Docker Compose
- **Orchestration**: Kubernetes 1.28+ with Helm charts (manifests/)
- **Monitoring**: Prometheus, Grafana, Loki for observability
- **Service Mesh**: Istio 1.20 for traffic management

## Key Development Patterns

### Authentication Flow
- Users authenticate through OAuth2/OpenID Connect via `openframe-api`
- JWTs are stored in HTTP-only cookies for security (moved from Authorization headers)
- Gateway converts cookies to Authorization headers for internal services
- Agents use separate authentication flow with service accounts

### Data Flow
1. **Ingestion**: External tools → `openframe-stream` → Kafka topics
2. **Processing**: Kafka → Stream Processing Service → enriched data → Cassandra/Pinot
3. **API Layer**: GraphQL queries → MongoDB/Cassandra/Pinot → client responses
4. **Real-time**: WebSocket connections through gateway for live updates

### Service Communication
- **External**: REST APIs through API Gateway
- **Internal**: Direct service-to-service HTTP calls
- **Async**: Kafka topics for event-driven communication
- **Configuration**: Spring Cloud Config for centralized configuration

## Project Structure

```
.
├── openframe/                          # Java services and libraries
│   ├── services/                       # Microservices
│   │   ├── openframe-gateway/          # API Gateway (port routing)
│   │   ├── openframe-api/              # GraphQL API and OAuth
│   │   ├── openframe-management/       # Admin and scheduled tasks

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flamingo-stack/openframe-oss-tenant](https://github.com/flamingo-stack/openframe-oss-tenant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
