---
trigger: always_on
description: Cursor rules that block deprecated, phantom, or incorrect NestJS imports, decorators, providers, modules, and testing patterns.
---

# NestJS Anti-Hallucination Rules

These rules OVERRIDE all other generation behavior. Check EVERY line of generated code against these rules.

## Banned Imports & Phantom Packages

### NEVER import these — they don't exist or are deprecated:
```
❌ @nestjs/core/decorators    — not a real export path
❌ @nestjs/swagger/decorators — import from @nestjs/swagger directly
❌ @nestjs/typeorm/repository — not a real export path
❌ @nestjs/passport/strategies — import from passport-jwt, passport-local, etc.
❌ @nestjs/bull/decorators     — import from @nestjs/bullmq (bull is legacy)
❌ nestjs-redis               — use @nestjs-modules/ioredis or ioredis directly
❌ @nestjs/cqrs/decorators    — import from @nestjs/cqrs directly
❌ nestjs-config              — use @nestjs/config (official)
❌ nestjs-pino/logger          — import from nestjs-pino directly
```

### Correct import paths:
```typescript
// ✅ Swagger
import { ApiTags, ApiOperation, ApiResponse } from '@nestjs/swagger';

// ✅ TypeORM
import { InjectRepository } from '@nestjs/typeorm';
import { Repository, DataSource } from 'typeorm';

// ✅ BullMQ (NOT Bull)
import { InjectQueue, Processor, WorkerHost } from '@nestjs/bullmq';

// ✅ Config
import { ConfigService, ConfigModule } from '@nestjs/config';

// ✅ Passport
import { AuthGuard } from '@nestjs/passport';
import { Strategy as JwtStrategy, ExtractJwt } from 'passport-jwt';

// ✅ CQRS
import { CommandHandler, ICommandHandler, EventBus } from '@nestjs/cqrs';
```

## Deprecated Patterns — NEVER Generate These

### 1. getRepository() outside providers
```typescript
// ❌ DEPRECATED — removed in TypeORM 0.3+
const repo = getRepository(User);
const user = await getConnection().getRepository(User).find();

// ✅ CORRECT — inject via constructor
constructor(
  @InjectRepository(User)
  private readonly userRepo: Repository<User>,
) {}
```

### 2. @nestjs/bull (use @nestjs/bullmq)
```typescript
// ❌ OLD — @nestjs/bull with @Process decorator
import { Process, Processor } from '@nestjs/bull';
@Processor('queue')
class MyProcessor {
  @Process() async handle(job: Job) {}
}

// ✅ CURRENT — @nestjs/bullmq with WorkerHost
import { Processor, WorkerHost } from '@nestjs/bullmq';
@Processor('queue')
class MyProcessor extends WorkerHost {
  async process(job: Job): Promise<void> {}
}
```

### 3. Express-specific middleware mistakes
```typescript
// ❌ WRONG — Express req/res types in NestJS
import { Request, Response } from 'express';
@Get()
async findAll(@Req() req: Request, @Res() res: Response) {
  res.json(data); // Bypasses interceptors, serialization, exception filters
}

// ✅ CORRECT — Use NestJS decorators, return values
@Get()
async findAll(@Query() query: FindAllQueryDto): Promise<UserResponseDto[]> {
  return this.usersService.findAll(query);
}

// Only use @Res() when streaming files or SSE — add { passthrough: true }
@Get('download')
async download(@Res({ passthrough: true }) res: Response) {
  res.set('Content-Type', 'application/octet-stream');
  return new StreamableFile(stream);
}
```

### 4. Wrong decorator combinations
```typescript
// ❌ WRONG — @Injectable() on a controller
@Injectable()
@Controller('users')
export class UsersController {}

// ❌ WRONG — @Controller() on a service
@Controller()
@Injectable()
export class UsersService {}

// ❌ WRONG — @Body() in a GET handler
@Get()
async findAll(@Body() body: any) {} // GET requests should not have a body

// ❌ WRONG — Both @Param and @Query with same name
@Get(':id')
async findOne(@Param('id') paramId: string, @Query('id') queryId: string) {}
```

### 5. class-validator / class-transformer mistakes
```typescript
// ❌ WRONG — Validation without enabling in main.ts
// (AI often forgets this critical line)
// main.ts MUST have:
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,            // Strip unknown properties
  forbidNonWhitelisted: true, // Throw on unknown properties
  transform: true,            // Auto-transform payloads to DTO instances
  transformOptions: {
    enableImplicitConversion: true,
  },
}));

// ❌ WRONG — Mixing validation decorators with wrong transform
export class CreateUserDto {
  @IsString()
  name: string;

  @IsNumber()
  age: string; // Type mismatch! Decorator says number, type says string
}

// ✅ CORRECT — Types match decorators
export class CreateUserDto {
  @IsString()
  @IsNotEmpty()
  name: string;

  @IsInt()
  @Min(0)
  age: number;
}
```

### 6. Async module pitfalls
```typescript
// ❌ WRONG — useFactory without async when awaiting
TypeOrmModule.forRootAsync({
  useFactory: (config: ConfigService) => ({
    type: 'postgres',
    url: config.get('DATABASE_URL'), // Not awaited, no inject
  }),
})

// ✅ CORRECT
TypeOrmModule.forRootAsync({
  imports: [ConfigModule],
  inject: [ConfigService],
  useFactory: async (config: ConfigService) => ({
    type: 'postgres',
    url: config.getOrThrow<string>('DATABASE_URL'),
    autoLoadEntities: true,
    synchronize: false, // NEVER true in production
  }),
})
```

## Type Safety Rules

### NEVER generate `any`
```typescript
// ❌ BANNED
catch (error: any) { ... }
const data: any = await response.json();
private cache = new Map<string, any>();

// ✅ REQUIRED
catch (error: unknown) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XD3an/awesome-ai-coding-all-in-one](https://github.com/XD3an/awesome-ai-coding-all-in-one) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
