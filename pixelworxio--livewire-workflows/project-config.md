---
trigger: always_on
description: This document provides AI assistants with comprehensive information about the Livewire Workflows codebase structure, development workflows, and key conventions.
---

# CLAUDE.md - AI Assistant Guide for Livewire Workflows

This document provides AI assistants with comprehensive information about the Livewire Workflows codebase structure, development workflows, and key conventions.

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture & Design Patterns](#architecture--design-patterns)
3. [Directory Structure](#directory-structure)
4. [Core Components](#core-components)
5. [Development Workflow](#development-workflow)
6. [Coding Standards](#coding-standards)
7. [Testing Conventions](#testing-conventions)
8. [Key Concepts](#key-concepts)
9. [Common Tasks](#common-tasks)
10. [Troubleshooting](#troubleshooting)

---

## Project Overview

**Livewire Workflows** is a Laravel package that enables developers to build multi-step workflows with zero boilerplate. It provides an expressive DSL for defining complex user journeys (onboarding, checkouts, surveys) with automatic route registration, guard-based navigation, state persistence, and full Livewire integration.

### Key Information

- **Package Name**: `pixelworxio/livewire-workflows`
- **Namespace**: `Pixelworxio\LivewireWorkflows`
- **Requirements**: PHP 8.3+, Laravel 11.x/12.x/13.x, Livewire 3.x/4.x
- **License**: MIT
- **Test Framework**: Pest v3/v4
- **Code Quality**: PHPStan (level 2), Laravel Pint

### Primary Features

1. **Auto-generated routes** from DSL definitions
2. **Guard-based navigation** with positive semantics
3. **State persistence** (session, database, or null)
4. **History tracking** for back navigation
5. **Progress tracking** API
6. **Event-driven extensibility**
7. **Livewire 3 native integration**

---

## Architecture & Design Patterns

### Design Patterns Used

#### 1. Fluent Builder Pattern
- **Classes**: `FlowBuilder`, `StepBuilder`
- **Purpose**: Provide expressive DSL for workflow definition
- **Result**: Immutable DTOs (`WorkflowDefinition`, `StepDefinition`)

```php
Workflow::flow('onboarding')
    ->entersAt(name: 'onboarding.start', path: '/onboarding')
    ->finishesAt('dashboard')
    ->step('verify-email')
        ->goTo(VerifyEmail::class)
        ->unlessPasses(EmailVerifiedGuard::class)
        ->order(10);
```

#### 2. Repository Pattern
- **Contract**: `WorkflowStateRepository`
- **Implementations**: `SessionWorkflowStateRepository`, `EloquentWorkflowStateRepository`, `NullStateRepository`
- **Purpose**: Abstract state persistence mechanism

#### 3. Strategy Pattern
- **Usage**: Swap state repositories via configuration
- **Binding**: Service container resolves correct implementation

#### 4. Pipeline Pattern
- **Location**: `WorkflowEngine`
- **Purpose**: Sequential guard evaluation using Laravel's Pipeline

#### 5. Facade Pattern
- **Class**: `Workflow` facade
- **Target**: `WorkflowRegistrar`
- **Purpose**: Static access to workflow registration

#### 6. Registry Pattern
- **Class**: `WorkflowRegistrar`
- **Purpose**: Central registry of all workflows with lazy finalization

#### 7. DTO Pattern
- **Classes**: `WorkflowDefinition`, `StepDefinition`
- **Characteristics**: Immutable, validated on construction

#### 8. Trait Composition
- **Trait**: `InteractsWithWorkflows`
- **Purpose**: Reusable functionality for Livewire components

#### 9. Observer Pattern (Events)
- **Events**: `WorkflowAdvanced`, `WorkflowCompleted`, `WorkflowStateClearing`
- **Purpose**: Extensibility and analytics hooks

---

## Directory Structure

```
livewire-workflows/
├── config/
│   └── livewire-workflows.php          # Package configuration
├── database/
│   └── migrations/                     # Database migration stubs
├── resources/                          # Package resources (images, etc.)
├── src/
│   ├── Attributes/                     # PHP 8 Attributes
│   │   ├── WorkflowName.php           # Workflow name declaration attribute
│   │   ├── WorkflowState.php          # State persistence attribute
│   │   └── WorkflowStep.php           # Step metadata attribute
│   ├── Commands/                       # Artisan commands
│   │   ├── MakeWorkflowCommand.php
│   │   ├── MakeWorkflowStepCommand.php
│   │   ├── MakeWorkflowGuardCommand.php
│   │   ├── WorkflowsInstallCommand.php
│   │   └── WorkflowsScanCommand.php
│   ├── Contracts/                      # Interfaces
│   │   ├── GuardContract.php
│   │   └── WorkflowStateRepository.php
│   ├── Events/                         # Laravel Events
│   │   ├── WorkflowAdvanced.php
│   │   ├── WorkflowCompleted.php
│   │   └── WorkflowStateClearing.php
│   ├── Exceptions/                     # Custom exceptions
│   │   ├── WorkflowNotFoundException.php
│   │   └── InvalidWorkflowConfigurationException.php
│   ├── Facades/                        # Laravel Facades
│   │   └── Workflow.php
│   ├── Http/Controllers/               # HTTP Controllers
│   │   └── WorkflowEntryController.php
│   ├── Livewire/Concerns/             # Livewire Traits
│   │   └── InteractsWithWorkflows.php
│   ├── Registrar/                      # DSL Implementation
│   │   ├── WorkflowRegistrar.php      # Central registry
│   │   ├── FlowBuilder.php            # Flow builder
│   │   └── StepBuilder.php            # Step builder

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pixelworxio/livewire-workflows](https://github.com/pixelworxio/livewire-workflows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
