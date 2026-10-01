---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Open Foris Collect is a multi-module Maven project for field-based inventory data collection and management. It provides a web-based survey design and data entry system with complex validation rules, multi-language support, and multiple data export formats.

## Build System

This is a Maven 3.6.3+ multi-module project requiring JDK 8+ and Node.js 12+.

### Build Commands

```bash
# Build all default modules
mvn clean install

# Build with installer modules (for creating distributable installers)
mvn clean install -Pinstaller

# Skip tests during build
mvn clean install -DskipTests

# Run tests only
mvn test

# Run specific test
mvn test -Dtest=ClassName#methodName
```

### Frontend Development

The React frontend is located in `collect-webapp/frontend/`:

```bash
cd collect-webapp/frontend

# Install dependencies
npm install

# Start development server (runs on port 3000)
npm start

# Build for production
npm run build
```

## Module Architecture

The project is organized into the following Maven modules:

### Core Modules (Default Build)

- **collect-base**: Foundational classes used across all Collect variants (including Collect Mobile)
- **collect-core**: Core business logic, survey model, validation engine, data persistence. Used by Collect Earth and Collect Mobile
- **collect-rdb**: Relational database export/generation functionality
- **collect-server**: Server-side REST API layer and business services for the web application
- **collect-webapp**: Client-side web application (React + Material-UI frontend, Spring/ZK backend)

### Installation Modules (Built with `-Pinstaller` profile)

- **collect-installation**: Parent module for installer-related components
  - **collect-assembly**: Application assembly for packaging
  - **collect-autoupdater**: Auto-update functionality
  - **collect-control-panel**: Desktop control panel application
  - **collect-installer**: InstallBuilder-based installer
  - **collect-updater**: Update package generation

## Technology Stack

### Backend
- Spring Framework 5.3.27 (beans, context, web MVC, security, transactions)
- Spring Security 5.8.3
- Liquibase for database migrations (changelogs in `collect-core/src/main/resources/org/openforis/collect/db/changelog/`)
- Supported databases: PostgreSQL, SQLite, H2
- Apache Commons libraries (lang3, io, vfs2, dbcp2)

### Frontend
- React 17.0.2 with React Router 6
- Redux with redux-thunk for state management
- Material-UI (MUI) v5 for UI components
- Formik for forms
- React Bootstrap & Reactstrap
- SockJS + STOMP for WebSocket communication
- Superagent for HTTP requests
- Chart.js for data visualization

### Build Tools
- Maven (backend)
- React Scripts (frontend)
- Gulp for SASS compilation

## Key Architectural Patterns

### Backend Layers

1. **Manager Layer** (`collect-core/.../manager/`): Business logic managers for entities (CodeListManager, SurveyManager, RecordManager, UserManager, etc.)
2. **Service Layer** (`collect-core/.../service/`): Application services (CollectCodeListService, CollectRecordFileService, etc.)
3. **Persistence Layer** (`collect-core/.../persistence/`): DAO layer and database access
4. **Command Layer** (`collect-core/.../command/`): Command pattern for record operations
5. **Event Layer** (`collect-core/.../event/`): Event-driven architecture for async operations
6. **Metamodel Layer** (`collect-core/.../metamodel/`): Survey schema and metadata definitions
7. **Model Layer** (`collect-core/.../model/`): Domain model (Record, Entity, Attribute, etc.)

### Frontend Structure

- **scenes/**: Page components (DashboardPage, SurveyDesignerPage, DataCleansingPage, MapPage, SaikuPage)
- **datamanagement/**: Data entry and record management UI
- **actions/**: Redux action creators
- **reducers/**: Redux reducers
- **services/**: API client services
- **model/**: Frontend domain models
- **common/**: Shared components and utilities

## Database Migrations

Database schema changes are managed via Liquibase changelogs located in:
- `collect-core/src/main/resources/org/openforis/collect/db/changelog/`

Separate changelogs exist for different database types:
- `db.changelog-standard.xml` (common changes)
- `db.changelog-postgresql.xml` (PostgreSQL-specific)
- `db.changelog-sqlite.xml` (SQLite-specific)

## Development Setup

### Running Locally with Eclipse/Tomcat

1. Import all modules as Maven projects
2. Add Tomcat 9+ to Server Runtime Environments
3. Add required JDBC drivers to Tomcat lib folder: `sqlite-jdbc-*.jar`, `postgresql-*.jdbc4.jar`
4. Configure datasource in Tomcat `server.xml` GlobalNamingResources:
   ```xml
   <Resource auth="Container"
       driverClassName="org.sqlite.JDBC"
       factory="org.apache.tomcat.dbcp.dbcp2.BasicDataSourceFactory"
       name="jdbc/collectDs"
       type="javax.sql.DataSource"
       url="jdbc:sqlite:${user.home}/OpenForis/CollectDev/data/collect.db" />
   ```
5. Add `devMode=true` parameter to Tomcat `context.xml`:
   ```xml

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openforis/collect](https://github.com/openforis/collect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
