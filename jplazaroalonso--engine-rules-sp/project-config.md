---
trigger: always_on
description: Golang Microservice Implementation Rule
---


# Golang Microservice Implementation Rule

## Overview

This rule provides comprehensive guidance for implementing enterprise-grade Golang microservices using hexagonal architecture, DDD principles, and complete testing strategies. Based on successful Rules Engine microservice implementations achieving production-ready quality.

## 1. Feedback Section

### Identified Issues in Generic Microservice Development:
- **Unclear Architecture**: Mixing business logic with infrastructure concerns
- **Incomplete Testing**: Missing behavioral tests and integration coverage
- **Poor API Design**: Inconsistent REST/gRPC interface definitions
- **Missing Observability**: Inadequate monitoring and tracing
- **Weak Error Handling**: Generic error responses without context
- **No Event Integration**: Missing domain event publishing/consuming

### Recommendations:
- Use hexagonal architecture for clear separation of concerns
- Implement comprehensive test pyramid with behavioral testing
- Define precise API contracts with OpenAPI and Protobuf
- Add distributed tracing and structured logging
- Create domain-specific error types with proper HTTP status mapping
- Integrate NATS for event-driven communication

## 2. Role and Context Definition

### Target Role: Backend Developer (Golang)
### Background Context:
- **Domain**: Business rules engine with high-performance requirements
- **Architecture**: Microservices with DDD bounded contexts
- **Technology Stack**: Go 1.21+, Gin/Echo, GORM, NATS, PostgreSQL, Redis
- **Performance**: <500ms response time, 1000+ TPS throughput
- **Deployment**: Kubernetes with Docker containers

## 3. Objective and Goals

### Primary Objective:
Create production-ready Golang microservice implementing specific business domain with hexagonal architecture, comprehensive testing, and enterprise-grade observability.

### Success Criteria:
- **Architecture**: Clean hexagonal architecture with clear boundaries
- **Testing**: >85% coverage with unit, integration, and behavioral tests
- **Performance**: Meet SLA requirements (response time, throughput)
- **APIs**: Complete OpenAPI 3.0 and Protobuf specifications
- **Observability**: Distributed tracing, structured logging, metrics
- **Security**: Input validation, authentication, authorization

## 4. Key Terms and Definitions

### Technical Terminology:
- **Hexagonal Architecture**: Ports and adapters pattern isolating business logic
- **Domain Entity**: Core business object with identity and behavior
- **Value Object**: Immutable object defined by attributes (Money, Email)
- **Repository**: Data access abstraction hiding persistence details
- **Use Case**: Application service orchestrating business operations
- **Domain Event**: Something meaningful that happened in the domain
- **Anti-Corruption Layer**: Translation layer for external system integration

## 5. Task Decomposition (Chain-of-Thought)

### Step 1: Project Structure Setup
- **Input**: Service name, domain requirements, API specifications
- **Process**: Create hexagonal architecture project structure with proper Go modules
- **Output**: Complete project skeleton with dependency injection setup
- **Human Validation Point**: Review project structure follows hexagonal architecture principles

### Step 2: Domain Layer Implementation
- **Input**: Domain requirements, entity definitions, business rules
- **Process**: Implement entities, value objects, domain services, and specifications
- **Output**: Pure domain layer with comprehensive unit tests
- **Human Validation Point**: Verify domain logic correctly implements business requirements

### Step 3: Application Layer Implementation
- **Input**: Use cases, domain layer, external service requirements
- **Process**: Create application services, command/query handlers, DTOs
- **Output**: Application services with input validation and orchestration
- **Human Validation Point**: Confirm use cases properly orchestrate domain operations

### Step 4: Infrastructure Layer Implementation
- **Input**: Database schemas, external API specifications, messaging requirements
- **Process**: Implement repositories, external adapters, event publishers/consumers
- **Output**: Infrastructure adapters with integration tests
- **Human Validation Point**: Validate infrastructure correctly implements port contracts

### Step 5: Interface Layer Implementation
- **Input**: API specifications (OpenAPI, Protobuf), authentication requirements
- **Process**: Create REST/gRPC controllers, middleware, request/response mapping
- **Output**: Complete API implementation with documentation
- **Human Validation Point**: Verify APIs match specifications and handle edge cases

### Step 6: Testing Implementation
- **Input**: All implemented layers, business scenarios, error conditions
- **Process**: Create comprehensive test suite following test pyramid
- **Output**: Complete test coverage with behavioral, integration, and unit tests
- **Human Validation Point**: Confirm tests cover all business scenarios and edge cases

### Step 7: Observability and Deployment
- **Input**: Service implementation, monitoring requirements, deployment specs
- **Process**: Add tracing, logging, metrics, health checks, Docker configuration
- **Output**: Production-ready service with observability and deployment artifacts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jplazaroalonso) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
