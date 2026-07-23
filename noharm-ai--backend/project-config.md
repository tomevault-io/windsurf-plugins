---
trigger: always_on
description: NoHarm is a healthcare/pharmacy clinical decision support system that helps prevent adverse drug events. This Flask-based backend provides APIs for prescription management, drug interaction checking, clinical interventions, patient monitoring, and regulatory compliance.
---

# NoHarm Backend - Agent Guide

## Overview

NoHarm is a healthcare/pharmacy clinical decision support system that helps prevent adverse drug events. This Flask-based backend provides APIs for prescription management, drug interaction checking, clinical interventions, patient monitoring, and regulatory compliance.

**Tech Stack:** Flask 3.0.3, SQLAlchemy 2.0.29, PostgreSQL, Redis, AWS (S3, SQS, Lambda)

## Architecture

### Application Structure

```
backend/
├── mobile.py                 # Application entrypoint
├── config.py                 # Configuration management
├── app/                      # Application factory and setup
│   ├── __init__.py          # Flask app factory (create_app)
│   ├── flask_config.py      # Environment-based configs
│   ├── extensions.py        # Flask extensions (db, jwt, cors, mail)
│   ├── blueprints.py        # Blueprint registration
│   ├── handlers.py          # Error handlers
│   ├── security.py          # Security headers
│   └── logging_config.py    # Logging configuration
├── models/                   # SQLAlchemy models (ORM)
│   ├── main.py              # Core models (User, Substance, etc.)
│   ├── prescription.py      # Prescription-related models
│   ├── regulation.py        # Regulatory models
│   ├── appendix.py          # Supporting/lookup models
│   ├── enums.py             # Enumerations
│   └── requests/            # Request validation models (Pydantic)
├── repository/               # Data access layer
├── services/                 # Business logic layer
│   ├── *_service.py         # Domain-specific services
│   ├── admin/               # Admin services
│   ├── regulation/          # Regulatory services
│   └── reports/             # Reporting services
├── routes/                   # API endpoint definitions (Flask blueprints)
│   ├── *_routes.py          # Route handlers
│   ├── admin/               # Admin routes
│   ├── regulation/          # Regulatory routes
│   └── reports/             # Report routes
├── decorators/               # Custom decorators
│   └── has_permission_decorator.py  # Permission checking
├── security/                 # Security utilities
├── utils/                    # Utility functions
├── exception/                # Custom exceptions
├── agents/                   # AI agent configurations
└── tests/                    # Test suite
```

### Design Patterns

**Layered Architecture:**
- **Routes Layer** (routes/): Handles HTTP requests/responses, parameter parsing, calls services
- **Services Layer** (services/): Contains business logic, orchestrates operations
- **Repository Layer** (repository/): Data access abstraction, database queries
- **Models Layer** (models/): SQLAlchemy ORM models, database schema

**Key Principles:**
- Application factory pattern for Flask app creation
- Repository pattern for data access
- Service layer for business logic separation
- Decorator-based authorization using `@has_permission`
- Schema-based multi-tenancy (each client has their own schema)

## Domain Concepts

### Core Entities

**Prescription Flow:**
- **Prescription**: Medical prescription with multiple drugs
- **PrescriptionDrug**: Individual drug in a prescription
- **Patient**: Patient information and history
- **Intervention**: Clinical pharmacist interventions on prescriptions
- **InterventionOutcome**: Results/outcomes of interventions

**Drug Information:**
- **Drug**: Drug catalog with attributes
- **Substance**: Active pharmaceutical ingredients
- **Outlier**: Statistical outliers for dose/frequency validation
- **DrugAttributes**: Schema-specific drug configurations

**Clinical Data:**
- **Exams**: Laboratory test results
- **ClinicalNotes**: Clinical observations and notes
- **Alerts**: Drug interaction, allergy, and protocol alerts

**Organizational:**
- **Segment**: Hospital units/departments
- **Department**: Department configuration
- **User**: System users (pharmacists, doctors, admins)
- **SchemaConfig**: Client-specific configurations

**Regulatory:**
- **RegSolicitation**: Regulatory approval requests
- **RegSolicitationDrug**: Drugs in regulatory requests

### Multi-Tenancy

NoHarm uses **PostgreSQL schema-based multi-tenancy**:
- Each client (hospital/organization) has a dedicated database schema
- Schema name is stored in JWT claims after authentication
- Schema is set per-request using SQLAlchemy execution options
- Public schema contains shared data (users, substances, global configs)

**Schema Switching:**
```python
db.session.connection(
    execution_options={"schema_translate_map": {None: schema_name}}
)
```

### Permission System

Role-based access control using decorators:
```python
@has_permission(Permission.READ_PRESCRIPTION, Permission.WRITE_PRESCRIPTION)
def some_service_function(user_permissions: list[Permission]):
    # Function receives user_permissions parameter
    # Can check permissions dynamically if needed
```

Permissions are defined in `decorators/has_permission_decorator.py`

## Database

### Primary Database
- **Type:** PostgreSQL

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [noharm-ai/backend](https://github.com/noharm-ai/backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
