---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenESPI-GreenButton-Java is a monorepo implementation of the NAESB Energy Services Provider Interface (ESPI) 4.0 specification for Green Button energy data standards. The project has been migrated to Java 25, Jakarta EE 10+, and Spring Boot 4.0.

## Build and Test Commands

### Build All Modules
```bash
# From repository root - builds all modules (common, datacustodian, thirdparty, authserver)
mvn clean install

# Build only core modules (common, datacustodian, thirdparty) - omits authserver
mvn clean install -Pspring-boot-only
```

### Run Tests
```bash
# Test all modules
mvn test

# Test specific module with dependencies
mvn test -pl openespi-datacustodian -am

# Test specific module only
cd openespi-common && mvn test

# Integration tests with TestContainers (requires Docker)
mvn verify -pl openespi-common -Pintegration-tests
```

### Run Applications
```bash
# Data Custodian (default: dev-mysql profile)
cd openespi-datacustodian && mvn spring-boot:run

# With specific profile
cd openespi-datacustodian && mvn spring-boot:run -Dspring-boot.run.profiles=dev-postgresql

# Authorization Server
cd openespi-authserver && mvn spring-boot:run

# Third Party
cd openespi-thirdparty && mvn spring-boot:run
```

### Run Single Test
```bash
# Run specific test class
mvn test -Dtest=UsagePointRepositoryTest

# Run specific test method
mvn test -Dtest=UsagePointRepositoryTest#testFindById

# Run test in specific module
mvn test -pl openespi-common -Dtest=UsagePointRepositoryTest
```

## Architecture

### Module Dependencies
The monorepo follows a layered architecture with these dependencies:
- **openespi-common** → Foundation library (no dependencies on other modules)
- **openespi-datacustodian** → Depends on openespi-common
- **openespi-authserver** → Independent module (NO dependency on openespi-common)
- **openespi-thirdparty** → Depends on openespi-common

IMPORTANT: The Authorization Server (openespi-authserver) is completely independent and does NOT depend on openespi-common. It has its own entity definitions, repositories, and services for OAuth2 authorization.

### Domain Model Structure
The domain model is organized in `openespi-common/src/main/java/org/greenbuttonalliance/espi/common/domain/`:
- **domain/common/** - Shared base entities (IdentifiedObject, LinkType, etc.)
- **domain/usage/** - ESPI energy usage entities (UsagePoint, MeterReading, IntervalBlock, ReadingType, RetailCustomer, etc.)
- **domain/customer/** - Customer entities from customer.xsd schema (CustomerAccount, CustomerAgreement, ServiceSupplier, Meter, etc.)

All ESPI domain entities extend `IdentifiedObject` which provides:
- UUID primary key (`id`)
- Self-reference URI (`selfLink`)
- Up-reference URI (`upLink`)
- XML marshalling support

### Service Layer Pattern
Services follow a consistent interface-implementation pattern:
- **Interfaces**: `openespi-common/src/main/java/org/greenbuttonalliance/espi/common/service/`
- **Implementations**: `openespi-common/src/main/java/org/greenbuttonalliance/espi/common/service/impl/`

Services are organized by domain:

**Usage Domain Services** (energy data from usage.xsd):
- `UsagePointService` - Energy usage point management
- `MeterReadingService` - Meter reading data
- `IntervalBlockService` - Time-series interval data
- `ReadingTypeService` - Reading type metadata
- `UsageSummaryService` - Usage aggregation summaries
- `ElectricPowerQualitySummaryService` - Power quality metrics
- `TimeConfigurationService` - Timezone and time parameters
- `RetailCustomerService` - Retail customer management (part of usage.xsd)

**Customer Domain Services** (from customer.xsd, in `service/customer/`):
- `CustomerAccountService` - Customer account management
- `CustomerService` - Customer information
- `MeterService` - Meter device information
- `ServiceLocationService` - Service location data
- `StatementService` - Billing statements

Note: Entities exist for CustomerAgreement and ServiceSupplier but services have not been implemented yet.

**Cross-Cutting Services**:
- `AuthorizationService` - OAuth2 authorization management
- `SubscriptionService` - Third-party subscriptions
- `ApplicationInformationService` - Application registration
- `NotificationService` - Event notifications
- `BatchListService` - Batch operations
- `DtoExportService` - Entity to DTO mapping and XML export

### REST API Structure
REST controllers are in `openespi-datacustodian/src/main/java/org/greenbuttonalliance/espi/datacustodian/web/`:
- **web/api/** - ESPI RESTful API endpoints (many currently `.disabled` during migration)
- **web/custodian/** - Data custodian-specific endpoints
- **web/customer/** - Retail customer portal endpoints

Note: Many REST controllers have `.disabled` extension during the Spring Boot 4.0 migration. They need to be re-enabled and tested after core functionality is validated.

### DTO and Mapping Layer
The project uses MapStruct for entity-to-DTO mappings:
- **DTOs**: `openespi-common/src/main/java/org/greenbuttonalliance/espi/common/dto/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GreenButtonAlliance) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
