---
trigger: always_on
description: Development workflow, common tasks, and adding new features to the users service
---


# Users Service - Development Workflow

## Adding a New Feature

Follow these steps to add new functionality while maintaining hexagonal architecture:

### 1. Define Domain Entity (if needed)

**Location**: `src/application/core/domain/`

If your feature requires a new entity or modifying existing ones:

```typescript
// src/application/core/domain/user.entity.ts
export class User {
  id: string;
  email: string;
  password?: string;
  name: string;
  city: string;
  uf: string;
  zipcode?: string;
  personType: string;
  avatarPath?: string;
}
```

Remember: Domain entities must be framework-agnostic!

### 2. Define Repository Port (if needed)

**Location**: `src/application/ports/out/`

Create or update the repository interface:

```typescript
// src/application/ports/out/users-repository.port.ts
export interface UserRepositoryPort {
  // Existing methods...

  // New method
  updateAvatar(id: string, avatarPath: string): Promise<User | null>;
}
```

### 3. Create Use Case

**Location**: `src/application/ports/in/`

Implement the business logic:

```typescript
// src/application/ports/in/user/updateUserAvatar.useCase.ts
import { Injectable, Inject } from '@nestjs/common';
import { UseCase } from '@/application/types/useCase.types';
import { Result, ResultFactory } from '@/application/types/result.types';
import { User } from '@/application/core/domain/user.entity';
import { UserRepositoryPort } from '@/application/ports/out/users-repository.port';
import { USERS_REPOSITORY } from '@/constants';
import { ErrorsEnum } from '@/application/core/errors/errors.enum';

export interface UpdateUserAvatarInput {
  userId: string;
  avatarPath: string;
}

@Injectable()
export class UpdateUserAvatarUseCase
  implements UseCase<UpdateUserAvatarInput, Promise<Result<User>>>
{
  constructor(
    @Inject(USERS_REPOSITORY)
    private readonly usersRepository: UserRepositoryPort,
  ) {}

  async execute(input: UpdateUserAvatarInput): Promise<Result<User>> {
    const user = await this.usersRepository.findById(input.userId);

    if (!user) {
      return ResultFactory.failure(ErrorsEnum.UserNotFound);
    }

    const updatedUser = await this.usersRepository.updateAvatar(
      input.userId,
      input.avatarPath,
    );

    if (!updatedUser) {
      return ResultFactory.failure(ErrorsEnum.UserNotFound);
    }

    return ResultFactory.success(updatedUser);
  }
}
```

### 4. Update Service

**Location**: `src/application/core/service/`

Add the new use case to the service:

```typescript
// src/application/core/service/users.service.ts
@Injectable()
export class UsersService {
  constructor(
    private readonly createUserUseCase: CreateUserUseCase,
    private readonly changePasswordUseCase: ChangePasswordUseCase,
    private readonly updateUserAvatarUseCase: UpdateUserAvatarUseCase, // New
  ) {}

  // Existing methods...

  async uploadAvatar(
    userId: string,
    avatarPath: string,
  ): Promise<Result<User>> {
    return this.updateUserAvatarUseCase.execute({ userId, avatarPath });
  }
}
```

### 5. Implement Repository

**Location**: `src/adapters/out/`

Implement the port interface:

```typescript
// src/adapters/out/users.repository.ts
@Injectable()
export class UsersRepository implements UserRepositoryPort {
  constructor(
    @InjectRepository(Users)
    private readonly usersRepository: Repository<Users>,
  ) {}

  // Existing methods...

  async updateAvatar(id: string, avatarPath: string): Promise<User | null> {
    const user = await this.usersRepository.findOneBy({ id });

    if (!user) {
      return null;
    }

    user.avatarPath = avatarPath;
    const savedUser = await this.usersRepository.save(user);

    return UserMapper.toDomain(savedUser);
  }
}
```

### 6. Add Controller Endpoint

**Location**: `src/adapters/in/`

Expose via HTTP:

```typescript
// src/adapters/in/user.controller.ts
@Controller('/users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  // Existing endpoints...

  @Post(':id/avatar')
  @UseInterceptors(FileInterceptor('avatar', uploadConfig))
  async uploadAvatar(
    @Param('id') id: string,
    @UploadedFile() file: Express.Multer.File,
  ) {
    const avatarPath = `temp/uploads/${file.filename}`;
    const result = await this.usersService.uploadAvatar(id, avatarPath);

    if (!result.isSuccess) {
      switch (result.error) {
        case ErrorsEnum.UserNotFound:
          throw new HttpException(result.error, HttpStatus.NOT_FOUND);
        default:
          throw new HttpException(result.error, HttpStatus.BAD_REQUEST);
      }
    }

    return result.value;
  }
}
```

### 7. Update Module

**Location**: `src/user.module.ts`

Register the new use case:

```typescript
@Module({
  imports: [
    /* ... */
  ],
  controllers: [UsersController],
  providers: [
    { provide: USERS_REPOSITORY, useClass: UsersRepository },
    { provide: DONOR_REPOSITORY, useClass: DonorRepository },
    { provide: COMPANY_REPOSITORY, useClass: CompanyRepository },
    CreateUserUseCase,
    ChangePasswordUseCase,
    UpdateUserAvatarUseCase, // Add this
    CreateDonorUseCase,
    CreateCompanyUseCase,
    UsersService,
  ],
})
export class AppModule {}
```

### 8. Write Tests

Create tests for each layer:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [c3ny/users-service](https://github.com/c3ny/users-service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
