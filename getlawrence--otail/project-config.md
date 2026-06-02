---
trigger: always_on
description: OTail is a comprehensive platform for managing OpenTelemetry tail sampling agents through an intuitive web interface. It provides visual policy configuration, real-time agent management, and seamless deployment capabilities for OpenTelemetry-based observability pipelines.
---

# OTail - OpenTelemetry Tail Sampling

## Project Overview

OTail is a comprehensive platform for managing OpenTelemetry tail sampling agents through an intuitive web interface. It provides visual policy configuration, real-time agent management, and seamless deployment capabilities for OpenTelemetry-based observability pipelines.

## Architecture

OTail follows a microservices architecture with the following core components:

### Core Services
- **Frontend (otail-web)**: React-based web UI for policy configuration and agent management
- **Backend (otail-server)**: Go-based API server handling business logic and agent communication
- **Collector (otail-col)**: OpenTelemetry collector with OpAMP integration for dynamic configuration
- **Database Layer**: ClickHouse for telemetry data storage, MongoDB for application data
- **Monitoring Stack**: Prometheus, Grafana, and Jaeger for observability

### Communication Flow
```
Applications → OTLP → Collector → ClickHouse
                    ↓
                OpAMP Server → Agent Management
                    ↓
                Web UI ← Backend API
```

## Tech Stack

### Frontend
- **Framework**: React 18 with TypeScript
- **Build Tool**: Vite
- **UI Components**: Radix UI primitives with Tailwind CSS
- **State Management**: React Context + Hooks
- **Code Editor**: Monaco Editor for YAML configuration
- **Testing**: Jest, Playwright, Storybook
- **WASM Integration**: Go-based OTTL evaluator compiled to WebAssembly

### Backend
- **Language**: Go 1.22+
- **Web Framework**: Chi router with CORS support
- **Database Drivers**: 
  - ClickHouse Go driver v2.30.0
  - MongoDB Go driver v2.0.0
- **Authentication**: JWT-based with bcrypt password hashing
- **OpAMP Integration**: OpenTelemetry OpAMP Go client v0.17.0
- **Observability**: OpenTelemetry SDK with OTLP exporters
- **Logging**: Structured logging with Zap

### Infrastructure
- **Containerization**: Docker with multi-stage builds
- **Orchestration**: Kubernetes with Helm charts
- **Databases**: 
  - ClickHouse for time-series telemetry data
  - MongoDB for application metadata and user management
- **Monitoring**: 
  - Prometheus for metrics collection
  - Grafana for visualization
  - Jaeger for distributed tracing
- **Networking**: Ingress-NGINX for external access

## Project Structure

```
OTail/
├── otail-web/                 # Frontend React application
│   ├── src/
│   │   ├── components/        # Reusable UI components
│   │   ├── pages/            # Application pages
│   │   ├── hooks/            # Custom React hooks
│   │   ├── lib/              # Utility libraries and policy evaluators
│   │   ├── api/              # API client and type definitions
│   │   └── contexts/         # React context providers
│   ├── wasm/                 # WebAssembly modules (OTTL evaluator)
│   └── public/               # Static assets
├── otail-server/             # Backend Go API server
│   ├── pkg/
│   │   ├── agents/           # Agent management and OpAMP integration
│   │   ├── auth/             # Authentication middleware
│   │   ├── organization/     # Organization management
│   │   ├── user/             # User management
│   │   └── telemetry/        # OpenTelemetry integration
│   └── main.go               # Application entry point
├── otail-col/                # OpenTelemetry collector
│   ├── Dockerfile            # Collector container definition
│   └── supervisor_docker.yml # OpAMP supervisor configuration
├── helm/                     # Kubernetes deployment charts
│   └── otail/               # Main Helm chart with all components
├── clickhouse/               # Database configuration and initialization
├── prometheus/               # Monitoring configuration
├── docker-compose.yml        # Local development environment
└── Makefile                  # Development workflow automation
```

## Prerequisites

### Development Environment
- **Docker & Docker Compose**: For local development
- **Node.js**: 18+ for frontend development
- **Go**: 1.22+ for backend development
- **Make**: For build automation (optional)

### Production Deployment
- **Kubernetes**: 1.19+ cluster
- **Helm**: 3.0+ package manager
- **Ingress Controller**: NGINX Ingress Controller
- **Storage**: Persistent volume support

## Quick Start

### Local Development

1. **Clone and Setup**
   ```bash
   git clone https://github.com/mottibec/otail.git
   cd otail
   make setup
   ```

2. **Start Development Environment**
   ```bash
   make dev
   ```

3. **Access Services**
   - Web UI: http://localhost:3000
   - API Server: http://localhost:8080
   - Grafana: http://localhost:3001
   - Prometheus: http://localhost:9090
   - Jaeger: http://localhost:16686
   - ClickHouse: http://localhost:8123

### Production Deployment

1. **Add Helm Repository**
   ```bash
   helm repo add otail https://mottibec.github.io/otail
   helm repo update
   ```

2. **Install OTail**
   ```bash
   helm install otail otail/otail --namespace otail --create-namespace
   ```

3. **Access the Application**
   - Configure your ingress controller

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getlawrence/OTail](https://github.com/getlawrence/OTail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
