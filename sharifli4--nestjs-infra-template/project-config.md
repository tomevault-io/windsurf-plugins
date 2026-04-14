---
trigger: always_on
description: This document provides strict rules for AI agents when working with configuration in this NestJS project.
---

# Configuration Management Rules for AI Agents

This document provides strict rules for AI agents when working with configuration in this NestJS project.

## 🎯 Core Principle

**Every domain has its own isolated configuration namespace. No domain can access another domain's configuration.**

---

## 📁 Configuration File Locations

### Rule 1: Infrastructure Config → `src/config/`

Place in `src/config/` if it's infrastructure-level:
- Database connection
- Redis connection  
- Logger settings
- JWT secrets
- API keys for infrastructure services

```
src/config/
├── env.ts                  # Environment accessor
├── database.config.ts      # Database config
├── redis.config.ts         # Redis config
├── logger.config.ts        # Logger config
└── jwt.config.ts           # JWT config
```

### Rule 2: Domain Config → `src/features/{domain}/config/`

Place in domain folder if it's domain-specific:
- Domain business rules
- Domain-specific API keys
- Domain feature flags
- Domain thresholds/limits

```
src/features/users/config/
└── users.config.ts

src/features/notifications/config/
└── notifications.config.ts

src/features/payments/config/
└── payments.config.ts
```

---

## 🏗️ Configuration File Structure

### Rule 3: All Configs Use `registerAs`

```typescript
import { registerAs } from '@nestjs/config';

export interface DomainConfig {
  // Type your configuration
  property1: string;
  property2: number;
}

export default registerAs(
  'domain-name',  // Unique namespace
  (): DomainConfig => ({
    property1: process.env.DOMAIN_PROPERTY1 || 'default',
    property2: parseInt(process.env.DOMAIN_PROPERTY2 || '0', 10),
  }),
);
```

### Rule 4: Configuration Template

```typescript
// src/features/users/config/users.config.ts
import { registerAs } from '@nestjs/config';

/**
 * Users domain configuration
 * Namespace: 'users'
 */
export interface UsersConfig {
  hashIdSalt: string;
  hashIdMinLength: number;
  maxLoginAttempts: number;
  lockoutDuration: number;
  sessionTimeout: number;
}

export default registerAs(
  'users',  // ✅ Unique namespace
  (): UsersConfig => ({
    hashIdSalt: process.env.USERS_HASHID_SALT || '',
    hashIdMinLength: parseInt(process.env.USERS_HASHID_MIN_LENGTH || '10', 10),
    maxLoginAttempts: parseInt(process.env.USERS_MAX_LOGIN_ATTEMPTS || '5', 10),
    lockoutDuration: parseInt(process.env.USERS_LOCKOUT_DURATION || '900', 10),
    sessionTimeout: parseInt(process.env.USERS_SESSION_TIMEOUT || '3600', 10),
  }),
);
```

**Required components:**
1. JSDoc comment with namespace
2. TypeScript interface
3. Export interface (for type checking)
4. Use `registerAs` with unique namespace
5. Environment variable names with domain prefix
6. Default values for non-critical configs
7. Type casting for numbers/booleans

---

## 🔑 Environment Variable Naming

### Rule 5: Use Domain Prefixes

Format: `{DOMAIN}_{PROPERTY_NAME}`

```bash
# ✅ CORRECT: Infrastructure (no prefix)
DB_HOST=localhost
DB_PORT=5432
REDIS_HOST=localhost
LOG_LEVEL=debug

# ✅ CORRECT: Domain-specific (with prefix)
USERS_HASHID_SALT=unique-salt
USERS_MAX_LOGIN_ATTEMPTS=5

NOTIFICATIONS_SMTP_HOST=smtp.example.com
NOTIFICATIONS_SMTP_PORT=587

PAYMENTS_STRIPE_API_KEY=sk_test_...
PAYMENTS_MAX_REFUND_DAYS=30
```

### Rule 6: Naming Convention

```bash
# Format:
{DOMAIN}_{FEATURE}_{PROPERTY}

# Examples:
USERS_PASSWORD_MIN_LENGTH=8
USERS_SESSION_TIMEOUT=3600
USERS_EMAIL_VERIFICATION_REQUIRED=true

PAYMENTS_STRIPE_API_KEY=sk_test_...
PAYMENTS_STRIPE_WEBHOOK_SECRET=whsec_...
PAYMENTS_REFUND_ENABLED=true

NOTIFICATIONS_EMAIL_ENABLED=true
NOTIFICATIONS_SMS_ENABLED=false
NOTIFICATIONS_RETRY_ATTEMPTS=3
```

---

## 📦 Registering Configuration

### Rule 7: Register in Module

```typescript
// src/features/users/users.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import usersConfig from './config/users.config';

@Module({
  imports: [
    ConfigModule.forFeature(usersConfig),  // ✅ Register domain config
  ],
  controllers: [UsersController],
  providers: [UsersService],
  exports: [UsersService],
})
export class UsersModule {}
```

### Rule 8: Infrastructure Config in app.module

```typescript
// src/app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';
import databaseConfig from './config/database.config';
import redisConfig from './config/redis.config';
import loggerConfig from './config/logger.config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,  // Makes ConfigService available globally
      load: [databaseConfig, redisConfig, loggerConfig],  // Infrastructure configs
    }),
    // ... feature modules
  ],
})
export class AppModule {}
```

---

## 🎯 Accessing Configuration

### Rule 9: Type-Safe Access

```typescript
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { UsersConfig } from './config/users.config';

@Injectable()
export class UsersService {
  private readonly config: UsersConfig;

  constructor(private readonly configService: ConfigService) {
    // ✅ CORRECT: Type-safe access to own domain config
    this.config = this.configService.get<UsersConfig>('users')!;
  }

  someMethod() {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sharifli4) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
