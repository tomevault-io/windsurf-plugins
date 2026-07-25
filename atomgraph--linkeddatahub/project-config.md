---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

LinkedDataHub (LDH) is a low-code Knowledge Graph application platform that enables managing data, creating visualizations, and building apps on RDF Knowledge Graphs. It's a completely data-driven platform where applications and documents are defined as data, managed using a single generic HTTP API, and presented using declarative technologies.

## Build System and Development Commands

LinkedDataHub uses Maven as the primary build system with Docker for containerization.

### Development Setup
```bash
# Initial setup (requires .env file configuration)
./bin/server-cert-gen.sh .env nginx ssl
docker-compose up --build
```

Service credentials (used by the entrypoint for Bearer auth) are stored in `secrets/credentials.trig`.

### Core Build Commands
```bash
# Maven build (Java 17 required)
mvn clean install

# Build specific profiles
mvn -Pstandalone clean install  # Standalone WAR
mvn -Pdependency clean install  # JAR dependency
mvn -Prelease clean install     # Release with signing

# Docker-based development
docker-compose up --build                    # Start all services
docker-compose down -v                       # Stop and remove volumes
sudo rm -rf data uploads && docker-compose down -v  # Complete reset
```

### Testing
```bash
# HTTP tests (requires running application)
cd http-tests
./run.sh ssl/owner/cert.pem [password] ssl/secretary/cert.pem [password]

# Test individual suites
find ./document-hierarchy/ -name '*.sh' -exec bash {} \;
```

## Architecture Overview

### Core Application Structure
- **JAX-RS based**: Uses Jersey framework for RESTful web services
- **Multi-application architecture**: Separate admin and end-user applications
- **Data-driven design**: Applications and resources defined as RDF data
- **XSLT-based UI**: Client-side rendering using Saxon-JS with XSLT transformations

### Key Components

#### Applications (`com.atomgraph.linkeddatahub.apps.model`)
- `AdminApplication` - Administrative interface and functions
- `EndUserApplication` - Main user-facing application
- Applications are data-driven and loaded from RDF datasets

#### Security & Authentication (`com.atomgraph.linkeddatahub.server.filter.request.auth`)
- WebID-based authentication with client certificates
- OAuth2 integration (Google)
- Authorization filters and context management
- Multi-level security: Agent, Authorization, and Application filters

#### Data Management (`com.atomgraph.linkeddatahub.model`)
- RDF-native data handling with Jena
- Import/Export functionality for CSV, RDF, and other formats
- SPARQL endpoint integration with separate admin and end-user stores

#### Resource Handling (`com.atomgraph.linkeddatahub.resource`)
- RESTful resource endpoints for CRUD operations
- File upload and content-addressed storage
- Transformation and generation utilities

#### Service Layer
- `ServiceContext` decouples HTTP infrastructure from `Service`, holding dataspace and service metadata separately
- Dataspace metadata and service metadata are split in configuration; types for `lapp:endUserApplication`/`lapp:adminApplication` are inferred on the fly from `system.trig`

### Dataspaces
Since v5.1.0, a single LDH instance supports multiple **dataspaces**, each identified by a distinct subdomain (origin). Each dataspace is a pair of applications: an end-user app (`<subdomain>`) and an admin app (`admin.<subdomain>`), routed by nginx via wildcard subdomain matching.

Configuration is split across two files:
- `config/dataspaces.trig` — public metadata: origins (`lapp:origin`), ontologies (`ldt:ontology`), stylesheets (`ac:stylesheet`)
- `config/system.trig` — internal wiring: maps apps to SPARQL services (`ldt:service`) and assigns types (`lapp:AdminApplication`/`lapp:EndUserApplication`)

Multiple dataspaces can share the same backend SPARQL service.

### Service Architecture
The application runs as a multi-container setup:
- **nginx**: Reverse proxy and SSL termination (wildcard subdomain routing for dataspaces)
- **linkeddatahub**: Main Java application (Tomcat)
- **fuseki-admin/fuseki-end-user**: Separate SPARQL stores
- **varnish-frontend/varnish-admin/varnish-end-user**: Caching layers

### Data Flow
1. Requests come through nginx proxy
2. Varnish provides caching layer
3. LinkedDataHub application handles business logic
4. RDF data is read/written via the **Graph Store Protocol** — each document in the hierarchy corresponds to a named graph in the triplestore; the document URI is the graph name
5. Data persisted to appropriate Fuseki triplestore
6. XSLT transforms data for client presentation

### Linked Data Proxy and Client-Side Rendering

LDH includes a Linked Data proxy that dereferences external URIs on behalf of the browser. The original design rendered proxied resources identically to local ones — server-side RDF fetch + XSLT. This created a DDoS/resource-exhaustion vector: scraper bots routing arbitrary external URIs through the proxy would trigger a full server-side pipeline (HTTP fetch → XSLT rendering) per request, exhausting HTTP connection pools and CPU.

The current design splits rendering by request origin:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AtomGraph/LinkedDataHub](https://github.com/AtomGraph/LinkedDataHub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
