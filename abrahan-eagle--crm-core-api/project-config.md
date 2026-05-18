---
trigger: always_on
description: Backend API del sistema CRM especializado en gestión de préstamos comerciales, construido con NestJS 10 y TypeScript. El sistema gestiona el ciclo completo desde leads hasta comisiones, incluyendo contactos, empresas, aplicaciones de préstamo, bancos y ofertas.
---

# CRM Core API - Contexto para AI

## Descripción General
Backend API del sistema CRM especializado en gestión de préstamos comerciales, construido con NestJS 10 y TypeScript. El sistema gestiona el ciclo completo desde leads hasta comisiones, incluyendo contactos, empresas, aplicaciones de préstamo, bancos y ofertas.

## Arquitectura

### Stack Tecnológico
- **Framework**: NestJS 10.0.5
- **Lenguaje**: TypeScript 4.9.5
- **Base de Datos**: MongoDB 7.6.2 (Mongoose)
- **Autenticación**: Auth0 (express-oauth2-jwt-bearer)
- **Cloud Storage**: AWS S3
- **Email**: AWS SES
- **VoIP**: CloudTalk
- **Notificaciones**: NotificationAPI
- **Patrón**: Clean Architecture + CQRS

### Estructura de Carpetas
- `src/app/`: Capa de Aplicación (Command Handlers, Query Handlers, DTOs, Services)
- `src/domain/`: Capa de Dominio (Entities, Commands, Queries, Repository Interfaces, Events)
- `src/infra/`: Capa de Infraestructura (REST Controllers, MongoDB Adapters, External Adapters, DI)

### Patrones Arquitectónicos

#### 1. Clean Architecture / Hexagonal Architecture
- **Domain Layer (Inner)**: Entidades puras, interfaces de repositorios, lógica de negocio, independiente de frameworks
- **Application Layer (Middle)**: Command/Query Handlers, DTOs, Application Services, Event Handlers, orquesta casos de uso
- **Infrastructure Layer (Outer)**: REST Controllers, MongoDB implementations, External adapters, implementa detalles técnicos
- **Principio**: Las capas internas no dependen de las externas

#### 2. CQRS (Command Query Responsibility Segregation)
- **Commands**: Modifican estado (Create, Update, Delete) - `domain/*/commands/` y `app/*/commands/`
- **Queries**: Solo leen datos (Get, Search, List) - `domain/*/queries/` y `app/*/queries/`
- **Ventajas**: Optimización independiente, escalabilidad, separación clara

#### 3. Repository Pattern
- Interfaces en Domain Layer (`domain/*/repositories/`)
- Implementaciones en Infrastructure Layer (`infra/adapters/mongo/*/repositories/`)
- **Ventajas**: Independencia de BD, testabilidad, flexibilidad

#### 4. Event-Driven Architecture
- Domain Events en `domain/*/events/`
- Event Handlers en `app/*/events/`
- Ejemplo: `ApplicationAcceptedEvent` → Crea Commission automáticamente
- **Ventajas**: Desacoplamiento, extensibilidad, trazabilidad

#### 5. Dependency Injection (NestJS)
- Módulos NestJS por feature
- Providers registrados en módulos
- Guards y Filters globales
- **Ventajas**: Testabilidad, flexibilidad, mantenibilidad

#### 6. Module Pattern (Feature-Based)
- Organización por features (Application, Bank, Contact, etc.)
- Cada feature tiene su módulo NestJS
- **Ventajas**: Alta cohesión, bajo acoplamiento, escalabilidad

### Separación de Responsabilidades

#### Domain Layer (Núcleo)
- **Responsabilidad**: Lógica de negocio pura, independiente de tecnología
- **Componentes**: Entities, Value Objects, Domain Events, Repository Interfaces, Domain Services, Business Rules
- **Características**: No depende de frameworks, no depende de infraestructura, testeable sin infraestructura

#### Application Layer (Orquestación)
- **Responsabilidad**: Orquestar casos de uso, coordinar entre Domain y Infrastructure
- **Componentes**: Command Handlers, Query Handlers, DTOs, Application Services, Event Handlers
- **Características**: Depende de Domain, orquesta flujos, transforma DTOs ↔ Entities

#### Infrastructure Layer (Adaptadores)
- **Responsabilidad**: Implementar detalles técnicos, adaptar sistemas externos
- **Componentes**: REST Controllers, MongoDB Adapters, External Adapters, Mappers, Middlewares
- **Características**: Depende de Application y Domain, implementa interfaces, maneja detalles técnicos

#### Cross-Cutting Concerns
- **Componentes**: Guards (PermissionsGuard), Middlewares (Auth, Logging), Exception Filters, Interceptors
- **Características**: Aplicados globalmente, no pertenecen a una capa específica

### Decisiones de Diseño

#### Clean Architecture vs MVC
- **Decisión**: Clean Architecture
- **Razón**: Independencia, testabilidad, mantenibilidad
- **Trade-off**: Complejidad inicial mayor

#### CQRS vs CRUD
- **Decisión**: CQRS
- **Razón**: Optimización independiente, escalabilidad
- **Trade-off**: Más código, dos flujos diferentes

#### MongoDB vs PostgreSQL
- **Decisión**: MongoDB
- **Razón**: Flexibilidad, documentos anidados, escalabilidad horizontal
- **Trade-off**: Sin transacciones ACID complejas

#### Domain Events vs Direct Calls
- **Decisión**: Domain Events
- **Razón**: Desacoplamiento, extensibilidad
- **Trade-off**: Debugging más complejo

#### Repository Pattern vs Active Record
- **Decisión**: Repository Pattern
- **Razón**: Independencia, testabilidad
- **Trade-off**: Capa adicional, mappers necesarios

#### Feature-Based vs Layer-Based
- **Decisión**: Feature-Based
- **Razón**: Alta cohesión, bajo acoplamiento, escalabilidad
- **Trade-off**: Duplicación potencial

## Módulos Principales

### Application (Aplicaciones)
- **Propósito**: Gestión completa de solicitudes de préstamo comercial

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Abrahan-Eagle/crm-core-api](https://github.com/Abrahan-Eagle/crm-core-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
