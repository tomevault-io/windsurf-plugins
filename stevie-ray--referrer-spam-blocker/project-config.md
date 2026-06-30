---
trigger: always_on
description: Handles reading and processing the domain list from `src/domains.txt`.
---

# Agent Guide: Referrer Spam Blocker

## Project Overview

This is the **Referrer Spam Blocker** project - a PHP library that generates configuration files for various web servers (Apache, Nginx, IIS, uWSGI, Caddy, Varnish, HAProxy, Lighttpd) and Google Analytics segments to block referrer spam traffic. OpenLiteSpeed users should use the Apache .htaccess file as OpenLiteSpeed is Apache-compatible.

### Key Details

- **Language**: PHP 8.3+
- **Namespace**: `StevieRay\`
- **Main Purpose**: Generate blacklist configuration files from a domain list to prevent referrer spam
- **Output**: Configuration files for multiple web servers and Google Analytics exclusion lists

## Development Environment: Docker Compose

This project uses **Docker Compose** for local development with a PHP 8.3 CLI container.

### Docker Configuration

- **PHP Version**: 8.3-cli
- **Container Name**: `referrer-spam-blocker-php`
- **Database**: Not used
- **Composer**: Included in custom Dockerfile

### Quick Start

```bash
# Build and start containers
docker compose build
docker compose up -d

# Install dependencies
docker compose exec php composer install

# Generate config files
docker compose exec php composer generate
```

### Common Commands

```bash
# Start containers
docker compose up -d

# Stop containers
docker compose down

# Install dependencies
docker compose exec php composer install

# Update dependencies
docker compose exec php composer update

# Generate config files
docker compose exec php composer generate

# Run tests
docker compose exec php composer test

# Run all quality checks (PHPStan, PHPCS, Psalm, Tests)
docker compose exec php composer quality

# Access the container shell
docker compose exec php sh

# Rebuild containers (after Dockerfile changes)
docker compose build
```

## Project Structure

```
referrer-spam-blocker/
├── src/
│   ├── CLI/              # Command-line interface
│   ├── Config/           # Config generators for different server types
│   ├── Domain/           # Domain processing logic
│   ├── Service/          # File writing services
│   └── domains.txt       # Source domain list (9118+ domains)
├── tests/
│   ├── Unit/             # Unit tests
│   └── Integration/      # Integration tests
├── Dockerfile            # Custom PHP 8.3 + Composer image
├── docker-compose.yml    # Docker Compose configuration
├── composer.json         # PHP dependencies
├── phpcs.xml            # PHP CodeSniffer rules (PSR-12)
├── phpunit.xml          # PHPUnit configuration
├── run.php              # CLI entry point
└── Generated files:     # Output files (git-tracked)
    ├── .htaccess
    ├── referral-spam.conf
    ├── referral-spam.vcl
    ├── web.config
    └── google-exclude-*.txt
```

## Key Components

### Generator (`src/Generator.php`)
Main class that orchestrates config file generation for all server types.

### Config Generators (`src/Config/`)
Each server type has its own generator class:
- `ApacheConfigGenerator` → `.htaccess`
- `NginxConfigGenerator` → `referral-spam.conf`
- `VarnishConfigGenerator` → `referral-spam.vcl`
- `IISConfigGenerator` → `web.config`
- `CaddyConfigGenerator` / `CaddyV2ConfigGenerator` → Caddy configs
- `UwsgiConfigGenerator` → `referral_spam.res`
- `HAProxyConfigGenerator` → `referral-spam.haproxy`
- `LighttpdConfigGenerator` → `referral-spam.lighttpd.conf`
- `GoogleAnalyticsConfigGenerator` → `google-exclude-*.txt` (split files)

**Note:** OpenLiteSpeed users should use the Apache `.htaccess` file as OpenLiteSpeed is Apache-compatible.

### Domain Processor (`src/Domain/DomainProcessor.php`)
Handles reading and processing the domain list from `src/domains.txt`.

## Development Workflow

1. **Start Docker**: `docker compose up -d`
2. **Install dependencies**: `docker compose exec php composer install`
3. **Make changes** to source code
4. **Run tests**: `docker compose exec php composer test`
5. **Check code quality**: `docker compose exec php composer quality`
6. **Generate configs**: `docker compose exec php composer generate`
7. **Commit changes** including generated config files

## Testing

The project uses:
- **PHPUnit 12.4** for unit tests
- **PHPStan** (Level 8) for static analysis
- **PHP CodeSniffer 4.0** (PSR-12 standard) for code style
- **Psalm** for additional static analysis

Run all quality checks via Composer script:
```bash
docker compose exec php composer quality
```

This runs in sequence: PHPStan → PHPCS → Psalm → Tests

## Code Style

- **PSR-12** coding standard (enforced by PHP CodeSniffer)
- **PHP-CS-Fixer** available for additional code formatting
- PHP 8.3+ features (typed properties, enums, etc.)
- Strict types: `declare(strict_types=1);` in all files

Available code style tools:
```bash
# Check code style (PHPCS - primary tool)
docker compose exec php composer phpcs

# Auto-fix code style (PHPCS)
docker compose exec php composer phpcbf

# Check with PHP-CS-Fixer (optional)
docker compose exec php composer php-cs-fixer

# Auto-fix with PHP-CS-Fixer
docker compose exec php composer php-cs-fixer:fix
```

## Important Notes for Agents

1. **Always use `docker compose` commands** - PHP/Composer are containerized

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Stevie-Ray/referrer-spam-blocker](https://github.com/Stevie-Ray/referrer-spam-blocker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
