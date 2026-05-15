---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

The OSCAL Tools project consists of three main components:

1. **CLI** (`cli/`) - Java command-line tool for performing operations on [OSCAL](https://pages.nist.gov/OSCAL/) (Open Security Controls Assessment Language) and Metaschema content
2. **Back-end** (`back-end/`) - Spring Boot REST API that exposes OSCAL operations via HTTP endpoints
3. **Front-end** (`front-end/`) - Next.js web application providing a user-friendly interface for OSCAL operations

All components are built on top of [Metaschema Java Tools](https://github.com/metaschema-framework/metaschema-java) and [OSCAL Java Library](https://github.com/metaschema-framework/liboscal-java/).

## Project Structure

```
oscal-cli/
├── cli/                 # Original OSCAL CLI command-line tool
│   ├── src/            # Java source code for CLI
│   ├── pom.xml         # Maven POM for CLI module
│   └── spotbugs-exclude.xml
├── back-end/           # Spring Boot REST API
│   ├── src/            # Java source code for API
│   └── pom.xml         # Maven POM for back-end module
├── front-end/          # Next.js web application
│   ├── src/            # TypeScript/React source code
│   ├── public/         # Static assets
│   ├── package.json    # Node.js dependencies
│   └── next.config.ts  # Next.js configuration
├── docs/               # Documentation directory
├── pom.xml             # Parent Maven POM (aggregator)
├── Dockerfile          # Multi-stage Docker build
├── docker-compose.yml  # Docker Compose configuration
├── dev.sh              # Local startup (PostgreSQL via Docker, backend, frontend)
└── stop.sh             # Stop all servers
```

## Build and Deployment Policy

**CRITICAL: DO NOT BUILD THE APPLICATION**

The user handles all builds, compilations, and deployments. Your role is to:

✅ **DO**:
- Make code changes to source files
- Fix compilation errors by editing code
- Update test files to match new signatures
- Suggest what needs to be built (e.g., "The backend needs to be rebuilt")
- Inform the user when changes require a rebuild

❌ **DO NOT**:
- Run `mvn clean install` or any Maven build commands
- Run `npm run build` or any frontend build commands
- Run `./dev.sh` or any startup scripts
- Execute any build-related Bash commands
- Attempt to compile or package the application

**Your workflow**:
1. Make necessary code changes
2. Inform the user: "Changes complete. Please rebuild the backend/frontend."
3. Let the user handle the build process

## Documentation Guidelines

**IMPORTANT**: All documentation files created during development should be placed in the `docs/` directory.

### Documentation Best Practices

1. **Location**: Always create documentation files in `docs/` directory, not in project root
2. **Naming**: Use descriptive UPPERCASE names with hyphens (e.g., `FEATURE-NAME-GUIDE.md`)
3. **Format**: Use Markdown (.md) format for all documentation
4. **Content**: Include:
   - Date and status at the top
   - Clear problem/solution sections
   - Code examples where relevant
   - Testing results
   - Step-by-step guides for complex features

### Current Documentation

The `docs/` directory contains:
- **AUTHORIZATION-FEATURE-SUMMARY.md** - Complete guide to the authorization feature
- **GCP-DEPLOYMENT-SETUP.md** - Complete guide for setting up GCP deployment with GitHub Actions
- **GITHUB-SECRETS-SETUP.md** - Quick reference for configuring GitHub secrets for GCP deployment
- **HEALTH-CHECK-API.md** - Health check API documentation for monitoring integration
- **TEMPLATE-EDITOR-FIX.md** - Technical details on template editor fixes
- **VARIABLE-DETECTION-SUMMARY.md** - User guide for variable detection
- **VARIABLE-PATTERN-UPDATE.md** - Pattern matching updates for variables
- **JAVA_SPRING_UPGRADE_PLAN.md** - Java and Spring upgrade planning

When creating new features or fixing issues, document your work in the `docs/` folder so future developers can understand the implementation and decisions made.

## Build Commands

### Building All Components

```bash
# Build all Maven modules (CLI + back-end) from root
mvn clean install

# Build without running tests
mvn clean install -DskipTests

# Build only the CLI module
cd cli && mvn clean install

# Build only the back-end module
cd back-end && mvn clean install

# Build the front-end
cd front-end && npm ci && npm run build
```

### Running Tests

```bash
# Run all Maven tests (CLI + back-end)
mvn test

# Run CLI tests only
cd cli && mvn test -Dtest=CLITest

# Run back-end tests only
cd back-end && mvn test

# Run front-end tests
cd front-end && npm test
```

## Running Locally

### Option 1: Development Mode (Recommended for Development)

```bash
# From project root - starts PostgreSQL (via Docker), back-end, and front-end
./dev.sh

# Stop all servers
./stop.sh
```

This will start:
- **Back-end API**: http://localhost:8080/api
- **Front-end UI**: http://localhost:3000

### Option 2: Run CLI Only

After building with `mvn install` in the `cli/` directory:

```bash
# Run the CLI from the build output
cli/target/appassembler/bin/oscal-cli --help

# On Windows

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RegScale/oscal-hub](https://github.com/RegScale/oscal-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
