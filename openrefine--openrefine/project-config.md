---
trigger: always_on
description: This file provides important information about the OpenRefine project and repository for AI agents and developers.
---

# AGENTS.md - OpenRefine Project Information

This file provides important information about the OpenRefine project and repository for AI agents and developers.

## Project Overview

**OpenRefine** is a Java-based power tool that allows you to load data, understand it, clean it up, reconcile it, and augment it with data coming from the web. All from a web browser and the comfort and privacy of your own computer.

- **License**: BSD-3-Clause
- **Official Website**: https://openrefine.org
- **Documentation**: https://openrefine.org/docs
- **Community Forum**: https://forum.openrefine.org
- **Version**: 3.10-SNAPSHOT (as of this writing)

## Technology Stack

### Backend
- **Language**: Java
- **Minimum Java Version**: JDK 21
- **Maximum Java Version**: JDK 26
- **Build Tool**: Apache Maven 
- **Project Structure**: Multi-module Maven project

### Frontend
- **JavaScript Libraries**: jQuery, jQuery UI, Select2, Underscore.js
- **Internationalization**: @wikimedia/jquery.i18n
- **Build/Package Management**: Node.js 24+ and npm 11.16.0+
- **E2E Testing**: Cypress with Node.js 24

### Project Modules
- `modules/core` - Core OpenRefine functionality
- `modules/grel` - GREL (General Refine Expression Language)
- `main` - Main application and webapp
- `server` - Server components
- `extensions` - Extension modules (database, jython, pc-axis, wikibase)
- `packaging` - Distribution packaging
- `benchmark` - Performance benchmarks

## Build System

### Building OpenRefine

```bash
./refine build
```

This command:
- Compiles all Java code using Maven
- Builds the webapp frontend
- Prepares all modules and extensions
- Creates necessary artifacts

### Running OpenRefine

```bash
# On Mac OS and Linux
./refine

# On Windows
refine.bat
```

**Configuration Options:**
- `-c <path>` - Path to refine.ini file (default: ./refine.ini)
- `-d <path>` - Path to the data directory
- `-H <host>` - Expected host header value
- `-i <interface>` - Network interface to bind (default: 127.0.0.1)
- `-m <memory>` - JVM min and max memory heap size (default: 1400M)
- `-p <port>` - Port to listen on (default: 3333)
- `-v <level>` - Verbosity level [error,warn,info,debug,trace]
- `-w <path>` - Path to the webapp (default: main/webapp)
- `--debug` - Enable JVM debugging on port 8000
- `--jmx` - Enable JMX monitoring

### Other Build Commands

```bash
./refine clean              # Clean compiled classes
./refine test              # Run all tests
./refine extensions_test   # Run extension tests
./refine server_test       # Run server tests
./refine e2e_tests         # Run end-to-end tests
./refine lint              # Reformat source code according to conventions
./refine mac_dist <ver>    # Make MacOS binary distribution
./refine windows_dist <ver> # Make Windows binary distribution
./refine linux_dist <ver>  # Make Linux binary distribution
./refine dist <ver>        # Make all distributions
```

## Testing Infrastructure

### Unit Tests
- **Framework**: TestNG (Java)
- **Location**: Throughout the codebase in `src/test` directories
- **Run**: `./refine test` or `./refine server_test` or `./refine extensions_test`

### End-to-End Tests
- **Framework**: Cypress
- **Location**: `main/tests/cypress/`
- **Setup**: 
  ```bash
  cd main/tests/cypress
  npm i -g yarn
  yarn install
  ```
- **Run**: `./refine e2e_tests`
- **Browser**: Chrome (default in CI)
- **Configuration**: Uses environment variables like `CYPRESS_BROWSER`, `CYPRESS_SPECS`, `CYPRESS_GROUP`

### Testing in CI
The project uses GitHub Actions for continuous integration:
- **E2E Tests**: `.github/workflows/pull_request_e2e.yml` - Runs Cypress tests on pull requests
- **Server Tests**: `.github/workflows/pull_request_server.yml` - Runs server-side tests
- **CodeQL Analysis**: Security scanning for Java and JavaScript

## Development Workflow

### Code Contributions
1. Fork the repository
2. Create a branch named with the issue number and brief description
3. Make changes (avoid unrelated modifications)
4. Create unit and/or E2E tests for your changes
5. Run `./refine lint` before submitting (CI will fail if lint fails)
6. Ensure all tests pass
7. Submit a pull request for review

### Code Style and Formatting
- **Linting**: Run `./refine lint` to reformat code according to OpenRefine conventions
- **EditorConfig**: The repository includes `.editorconfig` for consistent formatting:
  - Charset: UTF-8
  - Line endings: LF
  - Indent: 4 spaces (2 for YAML, JSON, LESS, shell scripts)
  - Max line length: 120 characters
  - Java imports organized with specific layout rules
  - Insert final newline

### Important Files
- `pom.xml` - Root Maven project configuration
- `refine` / `refine.bat` - Main launch scripts
- `refine.ini` - Runtime configuration (can be created)
- `.editorconfig` - Code formatting rules
- `CONTRIBUTING.md` - Contribution guidelines
- `GOVERNANCE.md` - Project governance model

## Extension System

OpenRefine supports a plugin architecture for extending functionality. Extensions are located in the `extensions/` directory:
- **database** - Database import/export functionality
- **jython** - Python scripting support via Jython
- **pc-axis** - PC-Axis file format support

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenRefine/OpenRefine](https://github.com/OpenRefine/OpenRefine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
