---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**GBVR-IoT** is a VR panoramic IoT platform built on Webman (PHP async framework) that combines VR panoramic viewing with IoT device management, specifically for GB28181-2016 video surveillance systems. The platform supports VR-based home decoration visualization and integrated IoT device control.

### Key Technologies
- **Framework**: Webman (built on Workerman for async PHP)
- **Language**: PHP 8.2+
- **Database**: MySQL with Doctrine DBAL (not ORM)
- **Cache**: Redis (dao-cache, api_cache, gb_gateway)
- **IoT Protocol**: GB28181-2016 (Chinese video surveillance standard)
- **Streaming**: ZLMediaKit for RTSP/RTP streaming
- **Frontend**: Vue.js 2.x with photo-sphere-viewer for VR panoramas

## Development Commands

### Starting/Stopping Services

```bash
# Main API server (port 8886)
php start.php start           # Start foreground
php start.php start -d        # Start as daemon
php start.php stop
php start.php restart
php start.php status
php start.php reload          # Reload (graceful restart)

# GB28181 SIP server
./start_gb28181.sh           # Includes health checks and orchestration
./stop_gb28181.sh

# Route management
php webman route:list | grep -E "stats|zlm|device"
```

### Code Generation

```bash
# Generate business layer (Service + DAO)
php webman make:biz -i Goods                    # Basic
php webman make:biz -i Student -s students       # Specify table
php webman make:biz -i Coupon --namespace {plugin}\Business\Coupon

# Add additional DAO to existing Service
php webman make:biz-dao -i Product -d ProductCatalog

# Create VIP member
php webman make:vip

# Check company approvals
php webman company:check
```

### Database Operations

```bash
# Run migrations
bin/phpmig migrate

# System initialization (after fresh install)
php webman system:init
```

## Architecture

### Directory Structure

```
app/
├── admin/                    # Admin panel
│   ├── controller/          # Controllers (extend BaseController)
│   └── config/routes/       # Routes at /admin/* and /admin/api/*
├── api/                     # Public API
│   ├── v1/                 # General APIs (auth, products, VIP, IoT)
│   ├── v2/                 # GB28181-specific APIs
│   └── config/routes/      # Routes at /api/v1/* and /api/v2/*
├── command/                 # Console commands
├── middleware/              # Shared middleware
│   ├── admin/              # Admin auth middleware
│   └── security/           # Security/firewall middleware
└── AbstractController.php   # Base controller with getBiz(), createService()

CoreW/                        # Custom framework layer
├── Bfw.php                  # Service container (extends Pimple)
├── Business/                # Business services (auto-generated)
│   └── {Module}/
│       ├── Service/Impl/    # Service implementations
│       └── Dao/Impl/        # DAO implementations
├── Context/                 # Service providers, autoloader
├── Dao/                     # DAO proxy with caching
├── Provider/                # Service registration
└── Sdk/                     # External SDK wrappers (ZLMediaKit, AMap, etc.)

Gb28181Gateway/              # GB28181 SIP server (separate process)
config/                      # Configuration files
support/                     # Webman helper files
```

### Service Container (Bfw)

The framework uses Pimple as the DI container with custom autoloading:

```php
// Service alias pattern: "Module:ServiceName"
$service = $this->createService('User:UserService');      // Resolves to CoreW\Business\User\Service\Impl\UserServiceImpl
$service = $this->createService('System:SystemService');  // Resolves to CoreW\Business\System\Service\Impl\SystemServiceImpl

// Direct container access
$zlmClient = $this->getBiz()->offsetGet('zlm_sdk');
```

**Service Autoloading**: The `ContainerAutoloader` resolves service aliases using:
- Empty prefix → `CoreW\Business\`
- `{Prefix}` → Looks up in `$this->aliases`
- Final class: `{Namespace}\{Middle}\Service\Impl\{Name}Impl`

**IMPORTANT - Workerman Process Container Access**:
```php
// In Workerman processes (app/process/*), use Core::instance() instead of $this->getBiz()
protected function getBfw(): \CoreW\Bfw
{
    return Core::instance();
}

protected function getStreamProxyService(): StreamProxyService
{
    return $this->getBfw()->service('StreamProxy:StreamProxyService');
}
// ❌ WRONG: Core::$container['service'] (undefined static property)
// ✅ CORRECT: Core::instance()->service('service')
```

### Layered Architecture

1. **Controllers** (`app/*/controller/`): Handle HTTP, call services
2. **Services** (`CoreW/Business/*/Service/Impl/`): Business logic
3. **DAOs** (`CoreW/Business/*/Dao/Impl/`): Database access with caching
4. **Models**: Doctrine entities (not traditional ORM)
5. **Enum** (`CoreW/Business/*/Enums/`): Enumerations

### Route Organization

**Admin routes** (`app/admin/config/routes/index.php`):
- `/admin/auth/*` - Login/logout
- `/admin/api/system/*` - System monitoring APIs
- `/admin/gb28181/*` - GB28181 management UI routes

**API routes** (`app/api/config/routes/api.php`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wzj177/gbvr-iot](https://github.com/wzj177/gbvr-iot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
