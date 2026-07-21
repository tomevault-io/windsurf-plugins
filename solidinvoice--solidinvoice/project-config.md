---
trigger: always_on
description: This document provides comprehensive guidance for AI assistants working with the SolidInvoice codebase. It covers the architecture, conventions, workflows, and best practices to help you understand and effectively contribute to this project.
---

# SolidInvoice - AI Assistant Guide

This document provides comprehensive guidance for AI assistants working with the SolidInvoice codebase. It covers the architecture, conventions, workflows, and best practices to help you understand and effectively contribute to this project.

## Table of Contents

- [Project Overview](#project-overview)
- [Technology Stack](#technology-stack)
- [Codebase Structure](#codebase-structure)
- [Architecture & Design Patterns](#architecture--design-patterns)
- [Development Workflow](#development-workflow)
- [Testing Strategy](#testing-strategy)
- [Code Quality & Standards](#code-quality--standards)
- [Common Tasks & Commands](#common-tasks--commands)
- [Key Conventions](#key-conventions)
- [Database & ORM](#database--orm)
- [API Development](#api-development)
- [Frontend Development](#frontend-development)
- [Security Considerations](#security-considerations)

---

## Project Overview

**SolidInvoice** is a sophisticated open-source invoicing application designed for small businesses and freelancers. It provides comprehensive billing operations including:

- Client and contact management
- Quote creation and management
- Invoice generation and tracking (including recurring invoices)
- Online payment processing (multiple gateways via Payum)
- Tax and discount handling
- RESTful API for integrations
- Multi-channel notifications (email, SMS, webhooks)

**Current Version:** 2.3.11
**License:** MIT
**Primary Author:** Pierre du Plessis

### Current Status

The current branch is in the process of a UI rewrite from AdminLTE (Bootstrap 4) to Tabler (Bootstrap 5.3).
Any changes to templates or frontend code should be made in accordance with the new Tabler design system.

---

## Technology Stack

### Backend

| Component          | Technology              | Version                |
|--------------------|-------------------------|------------------------|
| Framework          | Symfony                 | 7.1+                   |
| Platform           | SolidWorx/Platform      | dev-main               |
| PHP                | PHP                     | 8.4+                   |
| ORM                | Doctrine ORM            | 2.15+                  |
| Migrations         | Doctrine Migrations     | 3.5+                   |
| Database           | MySQL/PostgreSQL/SQLite | MySQL 8.0+ recommended |
| API Framework      | API Platform            | 4.0+                   |
| PDF Generation     | mPDF                    | 8.0+                   |
| Money Handling     | moneyphp/money          | 4.5+                   |
| Task Scheduling    | Symfony Schedule        | 7.1+                   |
| Payment Processing | Payum                   | 1.7+                   |
| Workflow           | Symfony Workflow        | 7.1+                   |

### Frontend

| Component            | Technology               | Version |
|----------------------|--------------------------|---------|
| Build Tool           | Webpack (Symfony Encore) | 5.3+    |
| CSS Preprocessor     | Sass                     | 1.63+   |
| JavaScript Framework | Stimulus                 | 3.2+    |
| UI Framework         | Tabler (Bootstrap)       | 1.4+    |
| Icons                | FontAwesome              | 6.4+    |
| Package Manager      | Bun                      | 1.3+    |

### Development Tools

- **Static Analysis:** PHPStan (Level 6)
- **Coding Standards:** EasyCodingStandard (Symplify)
- **Code Refactoring:** Rector (PHP 8.4+)
- **Testing:** PHPUnit 10.4+, Mockery, Symfony Panther
- **Fixtures:** Foundry
- **CI/CD:** GitHub Actions

### Distribution

- **Docker:** Official SolidInvoice Docker images available on Docker Hub (solidinvoice/solidinvoice). Runs the FrankenPHP binary inside the container.
- **FrankenPHP:** Official recommended installation method. Application is built into a single binary, which contains PHP, all required extensions, the web server, and the application code.
- **Archive:** ZIP/TAR archives for manual installation (all assets pre-compiled)

---

## Codebase Structure

### Root Directory Layout

```text
/home/user/SolidInvoice/
├── assets/                 # Frontend JavaScript and SCSS
│   ├── controllers/        # Stimulus controllers
│   ├── img/                # Images and icons
│   ├── scss/               # Sass stylesheets
│   ├── controllers.json    # Stimulus config
│   └── core.ts             # Main application entry
├── bin/                    # Executables (console, phpunit, etc.)
├── config/                 # Symfony configuration
│   ├── packages/           # Package configurations
│   ├── routes/             # Route definitions
│   └── services.php        # Service container config
├── docker/                 # Docker configuration
├── frankenphp/             # FrankenPHP code and config (Go files and shell scripts to build and compile the binary)
├── migrations/             # Database migrations
├── public/                 # Web-accessible files
├── scripts/                # Shell scripts for various tasks
├── src/                    # Application source code (18 bundles)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SolidInvoice/SolidInvoice](https://github.com/SolidInvoice/SolidInvoice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
