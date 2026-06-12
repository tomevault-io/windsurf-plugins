---
trigger: always_on
description: Allyabase is the foundational ecosystem for Planet Nine, providing a complete microservices architecture with federated wiki integration, sessionless authentication, and MAGIC protocol support.
---

# Allyabase - Planet Nine Ecosystem

## Overview

Allyabase is the foundational ecosystem for Planet Nine, providing a complete microservices architecture with federated wiki integration, sessionless authentication, and MAGIC protocol support.

**Location**: `/allyabase/`

## Core Architecture

### 🌐 **Federated Wiki Proxy Routing (November 2025)**

Allyabase now supports routing all service traffic through Federated Wiki as a single entry point, matching production architecture.

#### Wiki Plugin: wiki-plugin-allyabase
**Location**: `/src/wiki/wiki-plugin-allyabase/`

The wiki plugin provides:
- **Service Proxy Routes**: All microservices accessible via `/plugin/allyabase/{service}/*`
- **Contract Management**: Covenant contract signing and viewing
- **Feed Management**: Dolores feed subscriptions
- **Inventory System**: BDO-based inventory management
- **Deployment Tools**: Service deployment and configuration

#### Proxy Route Mapping

All Planet Nine services are accessible through the wiki proxy:

```
/plugin/allyabase/julia/*        → julia:3000
/plugin/allyabase/continuebee/*  → continuebee:2999
/plugin/allyabase/pref/*         → pref:3002
/plugin/allyabase/bdo/*          → bdo:3003
/plugin/allyabase/joan/*         → joan:3004
/plugin/allyabase/addie/*        → addie:3005
/plugin/allyabase/fount/*        → fount:3006
/plugin/allyabase/dolores/*      → dolores:3007
/plugin/allyabase/minnie/*       → minnie:2525
/plugin/allyabase/aretha/*       → aretha:7277
/plugin/allyabase/sanora/*       → sanora:7243
/plugin/allyabase/covenant/*     → covenant:3011
/plugin/allyabase/glyphenge/*    → glyphenge:3010
/plugin/allyabase/linkitylink/*  → glyphenge:3010 (alias)
```

#### Implementation Details

**Server-side Plugin Entry**: `/src/wiki/wiki-plugin-allyabase/index.js`
```javascript
module.exports = {
  server: require('./server/server.js')
};
```

**Proxy Route Handler**: `/src/wiki/wiki-plugin-allyabase/server/proxy.js`
- Handles all `/plugin/allyabase/{service}/*` routes
- Proxies HTTP requests to backend services
- Preserves method (GET, POST, PUT, DELETE)
- Forwards headers and body
- Returns responses transparently

### 🐳 **Docker Test Environment**

**Location**: `/deployment/docker/`

#### Flexible Multi-Base Architecture

The Docker setup supports running multiple isolated allyabase instances simultaneously:

**Scripts**:
- `spin-up-bases.sh` - Start 1-3 test bases with configurable options
- `Dockerfile-flexible` - Multi-service container image
- `start-with-ports.sh` - Service startup with dynamic port mapping

**Port Mapping** (Test Environment):
- Base 1: Host ports 5111-5125 → Docker internal ports
- Base 2: Host ports 5211-5225 → Docker internal ports
- Base 3: Host ports 5311-5325 → Docker internal ports

Each base includes:
- All 14 microservices
- Federated Wiki on port 3333 (mapped to 5x24)
- Glyphenge on port 3010 (mapped to 5x25)
- Wiki plugin with proxy routes enabled

#### Usage Examples

```bash
# Start 3 bases with clean rebuild
./spin-up-bases.sh --clean --build

# Start with seeding on Base 1
./spin-up-bases.sh --seed --seed-base=1

# Start with prof service enabled
./spin-up-bases.sh --enable-prof

# Test wiki proxy on Base 1
curl http://localhost:5124/plugin/allyabase/fount/health
```

### 📦 **Microservices**

**Location**: `/deployment/{service}/`

#### Core Services
1. **Fount** (3006) - Authentication and experience/nineum management
2. **BDO** (3003) - Big Dumb Object storage
3. **Joan** (3004) - Identity management
4. **Julia** (3000) - Messaging and coordination
5. **Pref** (3002) - User preferences
6. **Continuebee** (2999) - Session continuity

#### Application Services
7. **Addie** (3005) - AI assistant and payment processing
8. **Sanora** (7243) - E-commerce and product management
9. **Dolores** (3007) - Content discovery and feeds
10. **Aretha** (7277) - Ticket and access management
11. **Covenant** (3011) - Contract management with SVG visualization
12. **Minnie** (2525) - Email service

#### Platform Services
13. **Glyphenge** (3010) - Server-side SVG rendering and link tapestries
14. **Prof** (3008) - Profile management (optional)

### 🔐 **Sessionless Authentication**

All services use cryptographic signature-based authentication:
- No passwords or sessions
- secp256k1 keypairs
- Message signing with timestamps
- Per-service authentication middleware

### ⚡ **MAGIC Protocol**

Cross-service operations coordinated through MAGIC spells:
- Centralized Fount authentication
- Multi-service workflows
- Experience granting
- Gateway rewards

See individual service CLAUDE.md files for available spells.

## Test Environment Configuration

### SDK Configuration

Client SDKs support test-wiki mode for wiki proxy routing:

**BDO SDK** (`bdo-js`):
```javascript
bdo.configure({
  env: 'test-wiki',
  base: 1  // Uses Base 1 wiki proxy (port 5124)
});
```

**Fount SDK** (`fount-js`):
```javascript
fount.configure({
  env: 'test-wiki',
  base: 2  // Uses Base 2 wiki proxy (port 5224)
});
```

**Addie SDK** (`addie-js`):
```javascript
addie.configure({
  env: 'test-wiki',
  base: 3  // Uses Base 3 wiki proxy (port 5324)
});
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [freyja-love-and-magic/allyabase](https://github.com/freyja-love-and-magic/allyabase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-12 -->
