---
trigger: always_on
description: Always implement the infrastructure layer following Clean Architecture principles with repositories, mappers, entities, and DTOs. The infrastructure layer provides concrete implementations of domain interfaces.
---


# Infrastructure Layer Architecture Rule

Always implement the infrastructure layer following Clean Architecture principles with repositories, mappers, entities, and DTOs. The infrastructure layer provides concrete implementations of domain interfaces.

## Infrastructure Layer Structure

The infrastructure layer must be organized in the following structure:

```
infrastructure/
├── database/              # Database implementations
│   ├── typeorm/          # TypeORM write repositories
│   │   ├── entities/     # TypeORM entities
│   │   ├── mappers/      # TypeORM mappers
│   │   └── repositories/ # TypeORM repository implementations
│   └── mongodb/          # MongoDB read repositories
│       ├── dtos/         # MongoDB DTOs
│       ├── mappers/       # MongoDB mappers
│       └── repositories/ # MongoDB repository implementations
└── {other-services}/      # Other infrastructure services (storage, external APIs, etc.)
```

## TypeORM Write Repositories

### Structure

TypeORM repositories implement write repository interfaces and handle persistence of aggregates to PostgreSQL. They must:

- Extend `BaseTypeormMasterRepository` or `BaseTypeormTenantRepository`
- Implement the domain write repository interface
- Use mappers to convert between entities and aggregates
- Use `@Injectable()` decorator

### Master Database Repository

For entities stored in the master database (no tenant isolation):

```typescript
import { BaseTypeormMasterRepository } from '@/shared/infrastructure/database/typeorm/base-typeorm/base-typeorm-master/base-typeorm-master.repository';
import { TypeormMasterService } from '@/shared/infrastructure/database/typeorm/services/typeorm-master/typeorm-master.service';
import { ExampleAggregate } from '@/example-context/example/domain/aggregates/example.aggregate';
import { ExampleWriteRepository } from '@/example-context/example/domain/repositories/example-write.repository';
import { ExampleTypeormEntity } from '@/example-context/example/infrastructure/database/typeorm/entities/example-typeorm.entity';
import { ExampleTypeormMapper } from '@/example-context/example/infrastructure/database/typeorm/mappers/example-typeorm.mapper';
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class ExampleTypeormRepository
  extends BaseTypeormMasterRepository<ExampleTypeormEntity>
  implements ExampleWriteRepository
{
  constructor(
    typeormMasterService: TypeormMasterService,
    private readonly exampleTypeormMapper: ExampleTypeormMapper,
  ) {
    super(typeormMasterService, ExampleTypeormEntity);
    this.logger = new Logger(ExampleTypeormRepository.name);
  }

  /**
   * Finds an example by id
   *
   * @param id - The id of the example to find
   * @returns The example if found, null otherwise
   */
  async findById(id: string): Promise<ExampleAggregate | null> {
    this.logger.log(`Finding example by id: ${id}`);
    const entity = await this.repository.findOne({
      where: { id },
    });

    if (!entity) {
      return null;
    }

    return this.exampleTypeormMapper.toDomainEntity(entity);
  }

  /**
   * Saves an example
   *
   * @param example - The example to save
   * @returns The saved example
   */
  async save(example: ExampleAggregate): Promise<ExampleAggregate> {
    this.logger.log(`Saving example: ${example.id.value}`);
    const entity = this.exampleTypeormMapper.toTypeormEntity(example);

    const savedEntity = await this.repository.save(entity);

    return this.exampleTypeormMapper.toDomainEntity(savedEntity);
  }

  /**
   * Deletes an example (soft delete)
   *
   * @param id - The id of the example to delete
   */
  async delete(id: string): Promise<void> {
    this.logger.log(`Soft deleting example by id: ${id}`);
    await this.repository.softDelete(id);
  }
}
```

### Tenant Database Repository

For entities stored in the master database with tenant isolation (using `tenantId` column):

```typescript
import { BaseTypeormTenantRepository } from '@/shared/infrastructure/database/typeorm/base-typeorm/base-typeorm-tenant/base-typeorm-tenant.repository';
import { TypeormMasterService } from '@/shared/infrastructure/database/typeorm/services/typeorm-master/typeorm-master.service';
import { TenantContextService } from '@/shared/infrastructure/services/tenant-context/tenant-context.service';
import { ExampleAggregate } from '@/example-context/example/domain/aggregates/example.aggregate';
import { ExampleWriteRepository } from '@/example-context/example/domain/repositories/example-write.repository';
import { ExampleTypeormEntity } from '@/example-context/example/infrastructure/database/typeorm/entities/example-typeorm.entity';
import { ExampleTypeormMapper } from '@/example-context/example/infrastructure/database/typeorm/mappers/example-typeorm.mapper';
import { Injectable, Logger, Scope } from '@nestjs/common';

@Injectable({ scope: Scope.REQUEST })
export class ExampleTypeormRepository
  extends BaseTypeormTenantRepository<ExampleTypeormEntity>
  implements ExampleWriteRepository
{
  constructor(
    typeormMasterService: TypeormMasterService,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sisques-labs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
