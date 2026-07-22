---
trigger: always_on
description: This document provides essential context for AI agents working on the mailserver-admin project.
---

# mailserver-admin - AI Agent Context

This document provides essential context for AI agents working on the mailserver-admin project.

## Project Purpose

**mailserver-admin** is a web-based administration interface for [docker-mailserver](https://github.com/jeboehm/docker-mailserver). It provides a comprehensive management interface for mail domains, users, aliases, DKIM settings, and fetchmail configurations.

### Core Features

- **Domain Management**: Add, edit, and delete mail domains
- **User Management**: Create, update, and remove mail users with password management
- **Alias Management**: Define mail aliases for email forwarding
- **DKIM Management**: Configure DKIM settings for email authenticity
- **Fetchmail Configuration**: Set up and manage Fetchmail for external email retrieval
- **OAuth2 Integration**: Secure authentication using OAuth2 providers
- **DNS Setup Wizard**: Interactive wizard for DNS record configuration
- **Dashboard**: Monitoring and statistics for Dovecot and Rspamd services

## Technical Stack

### Core Framework

- **PHP**: >= 8.4 (strict types enabled)
- **Symfony**: 7.4.x (Framework Bundle, Console, Form, Security, Twig, etc.)
- **EasyAdmin Bundle**: v4.27.8 (admin interface)
- **Doctrine ORM**: 3.6.1 (database abstraction)
- **Doctrine Migrations**: 3.7.0 (database versioning)

### Key Dependencies

- **HWIOAuthBundle**: v2.4.0 (OAuth2 authentication)
- **Predis**: v3.3.0 (Redis client)
- **Symfony Asset Mapper**: Modern asset management
- **Symfony Stimulus Bundle**: JavaScript framework integration
- **Symfony UX Chart.js**: Dashboard visualizations

### Development Tools

- **PHP CS Fixer**: v3.93.0 (code style)
- **PHPStan**: Level 6 static analysis
- **PHPUnit**: ^12.0 (testing)
- **Rector**: 2.3.4 (code refactoring)
- **devenv**: Reproducible development environment (Nix-based)

### Infrastructure

- **MySQL/MariaDB**: Database (via Doctrine)
- **Redis**: Caching and data synchronization
- **Caddy**: Web server (development)
- **PHP-FPM**: PHP execution (development)

## Architecture Overview

### Service Architecture

The application integrates with multiple external services:

1. **docker-mailserver** (via Dovecot)
   - User authentication and management
   - Mailbox operations
   - Health checks via `doveadm` HTTP API

2. **Rspamd** (Spam filtering)
   - Statistics and monitoring
   - Configuration management
   - Health checks via HTTP API

3. **Redis**
   - Data synchronization between admin interface and mailserver
   - Caching for performance
   - Runtime data storage

4. **MySQL**
   - Primary data storage for domains, users, aliases
   - Doctrine ORM for database operations

### Directory Structure

```
mailserver-admin/
├── src/                    # Application source code
│   ├── Command/           # Symfony console commands
│   ├── Controller/        # HTTP controllers (Admin, User, Autoconfig, Security)
│   ├── Entity/            # Doctrine entities (Domain, User, Alias, etc.)
│   ├── Form/              # Symfony form types
│   ├── Repository/        # Doctrine repositories
│   ├── Service/           # Business logic services
│   │   ├── DKIM/         # DKIM key management and configuration
│   │   ├── Dovecot/      # Dovecot integration services
│   │   ├── Rspamd/       # Rspamd integration services
│   │   ├── FetchmailAccount/  # Fetchmail configuration
│   │   ├── DnsWizard/    # DNS validation wizard
│   │   └── Security/     # Authentication and authorization
│   ├── Subscriber/        # Doctrine event subscribers
│   ├── Validator/         # Custom validation constraints
│   └── Twig/             # Twig extensions
├── tests/                 # Test suite
│   ├── Unit/             # Unit tests
│   └── Integration/      # Integration tests
├── config/                # Symfony configuration
├── migrations/            # Database migrations
├── templates/            # Twig templates
├── public/               # Web root
├── assets/               # Frontend assets (JavaScript, CSS)
└── bin/                  # Executable scripts
```

### Key Services

#### DKIM Services (`src/Service/DKIM/`)

- **KeyGenerationService**: Generates DKIM key pairs
- **DKIMStatusService**: Checks DKIM configuration status
- **Config/Manager**: Manages DKIM configuration files
- **Config/MapGenerator**: Generates configuration maps

#### Dovecot Services (`src/Service/Dovecot/`)

- **DoveadmHttpClient**: HTTP client for Dovecot admin API
- User management operations
- Health check integration

#### Rspamd Services (`src/Service/Rspamd/`)

- **RspamdControllerClient**: HTTP client for Rspamd API
- **RspamdStatsService**: Statistics and monitoring
- Health check integration

#### Security Services (`src/Service/Security/`)

- **OAuth User Provider**: OAuth2 user authentication
- Role management (ROLE_ADMIN, ROLE_DOMAIN_ADMIN, ROLE_USER)
- Password hashing and validation

## Development Environment

### Prerequisites

- **Nix**: Required for devenv
- **direnv**: Automatically loads development environment
- **Composer**: PHP dependency management

### Setup

1. **Start development environment**:

   ```bash
   devenv up
   ```

   This starts:
   - PHP 8.4 with Redis, PDO MySQL, Xdebug extensions
   - MySQL database server

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeboehm/mailserver-admin](https://github.com/jeboehm/mailserver-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
