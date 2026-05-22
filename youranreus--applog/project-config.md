---
trigger: always_on
description: - **框架**: NestJS 10.x + Fastify
---

# Backend (NestJS) 专用规则

## 技术栈
- **框架**: NestJS 10.x + Fastify
- **数据库**: TypeORM + MySQL2
- **验证**: class-validator + class-transformer
- **语言**: TypeScript 5.x (严格模式)
- **包管理**: pnpm workspace

## 后端 TypeScript 配置
- Target: ES2021
- Module: CommonJS
- 路径别名: `@/*` 映射到 `src/*`
- 装饰器支持: 启用 `experimentalDecorators` 和 `emitDecoratorMetadata`

## 快速参考

### 核心装饰器和工具（来自 @reus-able/nestjs）
```typescript
// 权限控制
import { AuthRoles, UserParams } from '@reus-able/nestjs';
@AuthRoles('user')    // 需要 user 权限
@AuthRoles('admin')   // 需要 admin 权限
@UserParams() user: UserJwtPayload  // 获取当前用户

// 日志记录
import { HLogger, HLOGGER_TOKEN } from '@reus-able/nestjs';
@Inject(HLOGGER_TOKEN) private logger: HLogger;

// 异常处理
import { BusinessException } from '@reus-able/nestjs';
throw new BusinessException('错误信息');
```

### 全局模块（已配置，直接使用）
- **LoggerModule**: 日志服务（通过 `HLOGGER_TOKEN` 注入）
- **ConfigModule**: 配置管理（通过 `ConfigService` 注入）
- **AuthGuard**: 权限守卫（自动检查 `@AuthRoles`）
- **ValidationPipe**: 参数校验（自动转换类型）
- **异常过滤器**: 统一错误响应格式

### 类型定义（来自 @reus-able/types）
```typescript
import type { UserJwtPayload } from '@reus-able/types';
```

## 项目基础设施（必读）

### 全局模块

**LoggerModule** - 来自 `@reus-able/nestjs`
```typescript
import { HLogger, HLOGGER_TOKEN } from '@reus-able/nestjs';

@Injectable()
export class YourService {
  @Inject(HLOGGER_TOKEN)
  private logger: HLogger;

  someMethod() {
    this.logger.log('message', YourService.name);
    this.logger.error('error', YourService.name);
  }
}
```

**ConfigModule** - 来自 `@nestjs/config`
- 已配置为全局模块
- 支持多环境配置：`.env.production.local`, `.env.development.local`, `.env.production`, `.env.development`, `.env`
- 通过依赖注入使用：
```typescript
import { ConfigService } from '@nestjs/config';

constructor(private config: ConfigService) {}

const value = this.config.get<string>('KEY_NAME', 'default_value');
```

**TypeORM + MySQL**
- 已配置全局 TypeORM 连接
- 新模块需要注册实体：
```typescript
@Module({
  imports: [TypeOrmModule.forFeature([YourEntity])],
})
```

### 全局守卫与拦截器（已配置）
- **AuthGuard**: 全局权限守卫，自动检查路由权限
- **TransformInterceptor**: 统一响应格式转换
- **AllExceptionsFilter & HttpExceptionFilter**: 统一异常处理
- **ValidationPipe**: 全局参数校验（支持自动类型转换）

### 权限控制装饰器

**@AuthRoles** - 路由权限控制
```typescript
import { AuthRoles, UserParams } from '@reus-able/nestjs';
import type { UserJwtPayload } from '@reus-able/types';

@Controller('api')
export class SomeController {
  // 需要 user 权限（普通用户）
  @Get('data')
  @AuthRoles('user')
  getData(@UserParams() user: UserJwtPayload) {
    return this.service.getData(user.id);
  }

  // 需要 admin 权限（管理员）
  @Post('admin/action')
  @AuthRoles('admin')
  adminAction() {
    return this.service.doAdmin();
  }

  // 不需要权限校验时，不使用装饰器
  @Get('public')
  publicData() {
    return this.service.getPublic();
  }
}
```

**权限等级说明**：
- 项目只有两种权限：`user`（普通用户）和 `admin`（管理员）
- 不需要权限校验的接口不要使用 `@AuthRoles` 装饰器
- 使用 `@UserParams()` 装饰器获取当前登录用户信息

**@UserParams** - 获取当前用户信息
```typescript
import { UserParams } from '@reus-able/nestjs';
import type { UserJwtPayload } from '@reus-able/types';

@Get('profile')
@AuthRoles('user')
getUserProfile(@UserParams() user: UserJwtPayload) {
  // user 包含: id, username, role 等信息
  return this.service.getProfile(user.id);
}
```

### 异常处理
优先使用 `BusinessException` 处理业务异常：
```typescript
import { BusinessException } from '@reus-able/nestjs';

if (!data) {
  throw new BusinessException('数据不存在');
}

// 也可以使用 NestJS 内置异常
throw new NotFoundException('资源未找到');
```

### API 版本控制
项目启用了 URI 版本控制，默认版本为 `VERSION_NEUTRAL` 和 `'1'`：
```typescript
import { Controller, VERSION_NEUTRAL } from '@nestjs/common';

@Controller({
  path: 'users',
  version: [VERSION_NEUTRAL, '1'],
})
export class UserController {}
```

## NestJS 开发规范

### 1. 模块组织
- 按功能模块划分代码（如 `user`, `post`, `auth` 等）
- 每个模块包含: `*.module.ts`, `*.controller.ts`, `*.service.ts`
- 使用 barrel exports（index.ts）统一导出
- 模块中使用实体时，必须在 Module 的 imports 中注册：`TypeOrmModule.forFeature([Entity])`

### 2. 控制器 (Controllers)
```typescript
import { Controller, Get, Post, Param, Body, VERSION_NEUTRAL } from '@nestjs/common';
import { AuthRoles, UserParams } from '@reus-able/nestjs';
import type { UserJwtPayload } from '@reus-able/types';

@Controller({
  path: 'users',
  version: [VERSION_NEUTRAL, '1'],
})
export class UserController {
  constructor(private readonly userService: UserService) {}

  // 公开接口，无需权限
  @Get('list')
  async getList(): Promise<UserResponseDto[]> {
    return this.userService.getList();
  }

  // 需要 user 权限
  @Get(':id')
  @AuthRoles('user')
  async findOne(
    @Param('id') id: string,
    @UserParams() user: UserJwtPayload,
  ): Promise<UserResponseDto> {
    return this.userService.findOne(id, user);
  }

  // 需要 admin 权限
  @Post('create')
  @AuthRoles('admin')
  async create(
    @Body() createDto: CreateUserDto,
    @UserParams() user: UserJwtPayload,
  ): Promise<UserResponseDto> {
    return this.userService.create(createDto, user);
  }
}
```
- **必须使用 API 版本控制**：`version: [VERSION_NEUTRAL, '1']`
- **权限控制**：根据接口需求使用 `@AuthRoles('user')` 或 `@AuthRoles('admin')`
- **获取当前用户**：使用 `@UserParams()` 装饰器
- 使用 DTO 进行请求/响应类型定义
- 控制器应保持轻量，业务逻辑放在 Service 中

### 3. 服务 (Services)
```typescript
import { Inject, Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [youranreus/Applog](https://github.com/youranreus/Applog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
