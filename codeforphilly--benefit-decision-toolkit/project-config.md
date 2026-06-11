---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Benefit Decision Toolkit (BDT) is a platform for creating benefit eligibility screeners using Decision Model and Notation (DMN) and Form-JS. The project consists of four main applications:

- **library-api**: Standalone Quarkus API that generates REST endpoints from DMN files (Kogito-based)
- **builder-api + builder-frontend**: Web application for creating and managing screeners (admin tool). Also deploys a public-facing screener evaluation interface (end-user tool).

The core concept: Subject matter experts can define eligibility rules using visual DMN decision tables, which automatically become REST APIs and interactive screeners without traditional software development.

## Common Commands

### Development Setup

```bash
# One-time setup with Devbox (recommended)
bin/install-devbox && devbox run setup

# Or without Devbox (requires manual dependency installation)
bin/setup
```

### Running Services

```bash
# Start all services in development mode (uses process-compose)
devbox services up

# Or manually with process-compose (if not using devbox)
process-compose

# Run library-api standalone
cd library-api && quarkus dev
# Serves at http://localhost:8083, Swagger UI at /q/swagger-ui

# Run builder services (requires Firebase emulators)
# Terminal 1: Start Firebase emulators
firebase emulators:start --project demo-bdt-dev --only auth,firestore,storage

# Terminal 2: Start builder-api
cd builder-api && quarkus dev
# Debug port: 5005

# Terminal 3: Start builder-frontend
cd builder-frontend && npm run dev
```

### Building

```bash
# Build specific API
cd builder-api && mvn clean package
cd library-api && mvn clean package

# Build frontend
cd builder-frontend && npm run build

# Clean rebuild (useful when DMN files change)
mvn clean compile
```

### Testing

```bash
# Run Java tests for an API
cd builder-api && mvn test

# Run library-api tests with Bruno (API testing tool)
cd library-api/test/bdt && bru run

# Frontend doesn't have test suites currently
```

## High-Level Architecture

### Multi-Application Structure

This is a monorepo containing four distinct applications that work together:

1. **library-api** (Kogito-based DMN → REST API generator)
   - Standalone Quarkus app using Kogito for automatic API generation
   - DMN files in `src/main/resources/` become REST endpoints
   - See `library-api/CLAUDE.md` for detailed documentation

2. **builder-api** (Quarkus REST API, ~3,300 LOC)
   - Admin backend for screener creation and management
   - Integrates Firebase (Auth, Firestore, Cloud Storage)
   - Uses KIE DMN (not Kogito) for manual DMN compilation and evaluation
   - Main packages: `controller`, `service`, `persistence`, `model`

3. **builder-frontend** (Solid.js, Form-JS Editor, DMN-JS)
   - Visual editor for creating benefit screeners
   - Features: Form editor, DMN decision editor, benefit configuration, preview, publish
   - Routes: `/` (home), `/project/:id` (editor), `/check/:id` (DMN editor)

### Data Flow Architecture

```
Builder Flow:
Admin → builder-frontend → builder-api → Firebase (Firestore + Storage)
                                      ↓
                            Compile DMN → Store compiled JAR
                                      ↓
                                Publish screener
```

### Key Architectural Patterns

**Separation of Concerns**:
- **Eligibility Checks**: Reusable DMN models (independent decision logic)
- **Benefits**: Configurations that reference one or more eligibility checks
- **Forms**: Separate schemas defining user input fields
- **Screeners**: Containers that combine forms + benefits + checks

**DMN Processing Differences**:
- **library-api**: Uses Kogito (automatic code generation at build time)
- **builder-api**: Uses KIE DMN directly (runtime compilation from XML)

**Storage Strategy**:
- **Metadata** (relationships, configs): Firestore NoSQL collections
- **Large artifacts** (DMN files, form schemas, compiled JARs): Google Cloud Storage
- **Reference data** (location lookups): Embedded SQLite databases

**Authentication**:
- **builder-api/builder-frontend**: Firebase Auth required (user ownership model). Specific endpoints starting with /published are publicly accessible.
- **library-api**: No authentication (standalone utility)

### Technology Stack

**Backend (All APIs)**:
- **builder-api**: Quarkus 3.23.0, Java 21, KIE DMN 10.0.0
- **library-api**: Quarkus 2.16.10, Java 17, Kogito 1.44.1

**Frontend**:
- **Framework**: Solid.js (reactive JavaScript framework)
- **Form Builder/Renderer**: Form-JS (BPMN.io)
- **DMN Editor**: DMN-JS + Kogito Tooling
- **Styling**: Tailwind CSS

**Infrastructure**:
- **Dev Environment**: Devbox (Nix-based) or Devcontainer
- **Process Management**: process-compose
- **Cloud Services**: Firebase (Auth, Firestore, Cloud Storage)
- **Database**: SQLite (embedded for reference data)

## Development Workflow

### Working with DMN Files

**In library-api** (Kogito):
1. Add/edit DMN file in `library-api/src/main/resources/`
2. Run `quarkus dev` - Kogito auto-generates REST endpoints
3. Check `/q/swagger-ui` for new endpoints

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CodeForPhilly/benefit-decision-toolkit](https://github.com/CodeForPhilly/benefit-decision-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
