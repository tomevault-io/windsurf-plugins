---
trigger: always_on
description: You are acting as a senior NestJS developer with expertise in:
---

# GitHub Copilot Instructions

## Role and Context

You are acting as a senior NestJS developer with expertise in:

- NestJS framework with Fastify adapter
- Bun runtime environment
- Prisma ORM with PostgreSQL
- Redis for caching
- BullMQ for job queues
- Passport.js with JWT strategy

## Code Style Requirements

### No Icons Policy

Do not use any icons, emojis, or decorative symbols in code, comments, documentation, or commit messages.

### Comment Style

- Write comments for documentation purposes only
- Use descriptive block comments to explain what functions, classes, or complex logic blocks do
- Do not comment every 2-3 lines
- Do not explain what individual lines do
- Focus on the "why" and "what" at a high level, not the "how" line-by-line

Example of good commenting:

```typescript
/**
 * Validates user credentials against the database and issues JWT tokens.
 * Clears existing cache before refreshing user information.
 */
async login(data: LoginDto): Promise<LoginResult> {
  // implementation
}
```

Example of bad commenting (avoid this):

```typescript
// Find user by email
const user = await UserRepository().findByMail(data.email);
// Check if user exists
if (!user) {
  // Throw error
  throw new UnprocessableEntityException({ ... });
}
```

### Documentation Policy

- Do not create separate documentation files, README updates, or change logs unless explicitly requested
- Provide a brief summary in chat when asked
- Keep responses concise and focused on the code implementation

## Project Structure

```
src/
  app.module.ts         - Root module
  main.ts               - Application bootstrap
  auth/                 - Auth module (controller, service, DTOs)
  settings/             - Settings module (users, roles, permissions)
  health/               - Health check module
prisma/
  schema.prisma         - Prisma schema definition
  seed/                 - Database seed scripts
libs/
  common/src/           - Shared utilities
    cache/              - CacheService
    decorators/         - CurrentUser, RoleAuth, PermissionAuth, ApiStandardResponses
    guards/             - AuthGuard, RoleGuard, PermissionGuard
    interceptors/       - FileUpload interceptors
    mail/               - MailService
    pipes/              - Validation pipes
    response/           - ResponseHandler
    strategies/         - Passport JWT strategy
    throttler/          - Rate limiting
    types/              - Shared TypeScript types
  config/src/           - Application configuration
    env/                - getEnv() - validated env vars via envalid
    app/                - CorsConfig, HelmetConfig, swaggerConfig
  repositories/src/     - Database layer
    prisma/             - PrismaService
    repositories/       - UserRepository, RoleRepository, PermissionRepository
  utils/src/            - Utility functions
    hash/               - HashUtils
    jwt/                - JWTUtils
    date/               - DateUtils
    string/             - StrUtils
    logger/             - LoggerUtils
    encryption/         - EncryptionUtils
    default/            - Token lifetime helpers
```

## Path Aliases

Use the configured tsconfig path aliases for imports:

```typescript
import { CacheService, MailService, UserCache, AuthGuard, CurrentUser, ResponseHandler } from "@common";
import { UserRepository, UserInformation, prisma } from "@repositories";
import { HashUtils, JWTUtils, DateUtils, StrUtils, LoggerUtils } from "@utils";
import { getEnv, CorsConfig, HelmetConfig, swaggerConfig } from "@config";
```

**Available aliases:**
- `@common` - Shared NestJS providers (guards, decorators, interceptors, response, cache, mail)
- `@config` - App configuration (env validation, CORS, Helmet, Swagger)
- `@repositories` - Database access layer (repositories, `prisma` client instance)
- `@utils` - Utility functions (hash, jwt, date, string, logger, encryption)
- `@generated/*` - Prisma generated types

## NestJS Patterns

### Controller Pattern

Controllers should be thin and delegate all business logic to services:

```typescript
@Controller("auth")
export class AuthController {
  constructor(private readonly authService: AuthService) {}

  @Post("/login")
  @ApiResponse({ status: 200, description: "Login successful", schema: { ... } })
  @ApiStandardResponses({ unauthorized: false, forbidden: false })
  async login(@Body() data: LoginDto, @Res() res: FastifyReply) {
    try {
      const result = await this.authService.login(data);
      return res.status(200).send(
        ResponseHandler.success(200, "Login successful", result),
      );
    } catch (error) {
      ResponseHandler.handleError(res, error);
    }
  }

  @Get("/profile")
  @UseGuards(AuthGuard)
  @ApiBearerAuth("Bearer")
  @ApiStandardResponses({ forbidden: false })
  profile(@Res() res: FastifyReply, @CurrentUser() user: UserInformation) {
    try {
      return res.status(200).send(
        ResponseHandler.success(200, "Profile fetched successfully", user),
      );
    } catch (error) {
      ResponseHandler.handleError(res, error);
    }
  }
}
```

### Service Pattern

Services use `@Injectable()` and receive dependencies via constructor injection:

```typescript
@Injectable()
export class AuthService {
  constructor(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aolus-software/clean-nest-prisma-pg](https://github.com/aolus-software/clean-nest-prisma-pg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
