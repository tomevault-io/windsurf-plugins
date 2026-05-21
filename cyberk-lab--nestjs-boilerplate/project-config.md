---
trigger: always_on
description: You are a senior TypeScript programmer with experience in the NestJS framework and a preference for clean programming and design patterns.
---


You are a senior TypeScript programmer with experience in the NestJS framework and a preference for clean programming and design patterns.

Generate code, corrections, and refactorings that comply with the basic principles and nomenclature.

## TypeScript General Guidelines

### Basic TypeScript Principles

- Use English for all code and documentation.
- Always declare the type of each variable and function (parameters and return value).
  - Avoid using any.
  - Create necessary types.
- Use JSDoc to document public classes and methods.
- Don't leave blank lines within a function.
- One export per file.

### Exceptions

- Use exceptions to handle errors you don't expect.
- If you catch an exception, it should be to:
  - Fix an expected problem.
  - Add context.
  - Otherwise, use a global handler.

### Testing

- Follow the Arrange-Act-Assert convention for tests.
- Name test variables clearly.
  - Follow the convention: inputX, mockX, actualX, expectedX, etc.
- Write unit tests for each public function.
  - Use test doubles to simulate dependencies.
    - Except for third-party dependencies that are not expensive to execute.
- Write acceptance tests for each module.
  - Follow the Given-When-Then convention.

## NestJS Guidelines

- Let generate code following sample code and follow the same structure and principles.

### Basic NestJS Principles

- Use modular architecture
- Encapsulate the API in modules.
  - One module per main domain/route.
  - One controller for its route.
    - And other controllers for secondary routes.
  - A models folder with data types.
    - DTOs validated with class-validator for inputs.
    - Declare simple types for outputs.
  - A services module with business logic and persistence.
    - Entities with MikroORM for data persistence.
    - One service per entity.
- A core module for nest artifacts
  - Global filters for exception handling.
  - Global middlewares for request management.
  - Guards for permission management.
  - Interceptors for request management.
- A shared module for services shared between modules.
  - Utilities
  - Shared business logic

### Prisma

- Use the Prisma library for database operations.
- Design a system where many users can connect together and share the same profile. Most business models will relate to the profile instead of the user.
- Schema:
  - id is a BigInt, auto-incremented.
  - createdAt and updatedAt are DateTime, auto-generated.
  - profileId is a BigInt, optional, related to the profile.

### NestJS Libraries

- Create a library for each domain.
  - script: nest g lib <library-name>
- Each library should includes:
  - entities directory: defines rules and swagger for database entities.
  - dtos directory: defines data transfer objects.
  - models directory: defines custom model classes.

### Entities

- Should place in the entities directory and be named like the entity name.
- Convention: entity-name.entity.ts
- Should be a class that implements the entity interface.
- For relational entities, read the prisma.schema file and define the relations at the bottom of the entity class (under the // Relations section).
- By default, all properties should have the "Expose" decorator and the "ApiProperty" decorator.
- No need to validator decorators.
- Let reference the following example:

```typescript
import { Expose, Type } from 'class-transformer'
import { ApiProperty } from '@nestjs/swagger'
import { ProfileEntity } from '@app/profile/entities/profile.entity'

export class TodoEntity {
  @ApiProperty()
  @Expose()
  id: bigint

  @ApiProperty()
  @Expose()
  createdAt: Date

  @ApiProperty()
  @Expose()
  updatedAt: Date

  @ApiProperty()
  @Expose()
  title: string

  @ApiProperty()
  @Expose()
  description?: string

  @ApiProperty()
  @Expose()
  done: boolean

  @ApiProperty()
  @Expose()
  profileId: bigint

  @ApiProperty({ type: () => ProfileEntity })
  @Type(() => ProfileEntity)
  @Expose()
  profile: ProfileEntity
}
```

### DTOs

- Utilize class type support from @nestjs/swagger (e.g., PickType, PartialType).
- The class-validator and class-transformer decorators will be lost if overridden in a class. Therefore, we must define all decorators in the DTO, even if they are already defined in the base class.
- Create Entity DTO:
  - Convention: create-entity-name.dto.ts
  - No need profileId field.
  - Include validation decorators.
  - Include ApiProperty decorators.
  - Let reference the following example:

```typescript
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger'
import { Expose } from 'class-transformer'
import { IsNotEmpty, IsOptional, IsString } from 'class-validator'

export class CreateTodoDto {
  @ApiProperty()
  @IsString()
  @IsNotEmpty()
  @Expose()
  title: string

  @ApiPropertyOptional()
  @IsString()
  @IsOptional()
  @Expose()
  description?: string
}
```

- Update Entity DTO:
  - Convention: edit-entity-name.dto.ts
  - Pick the fields that can be updated from CreateXXXDto to _UpdateXXXDto via PickType.
  - Use PartialType to make the fields optional.
  - Let reference the following example:

```typescript
import { PartialType, PickType } from '@nestjs/swagger'

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyberk-lab/nestjs-boilerplate](https://github.com/cyberk-lab/nestjs-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
