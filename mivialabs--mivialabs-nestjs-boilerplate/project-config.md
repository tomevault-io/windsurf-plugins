---
trigger: always_on
description: Command Query Responsibility Segregation (CQRS) separates read and write operations for better scalability and maintainability.
---

# CQRS Pattern Rules

## Overview
Command Query Responsibility Segregation (CQRS) separates read and write operations for better scalability and maintainability.

## Command Structure

### Command Definition
```typescript
import { ICommand } from '@nestjs/cqrs';

export class CreateUserCommand implements ICommand {
  constructor(
    public readonly email: string,
    public readonly name: string,
    public readonly phone?: string,
  ) {}
}
```

### Command Handler
```typescript
import { CommandHandler, ICommandHandler } from '@nestjs/cqrs';
import { Injectable } from '@nestjs/common';

@Injectable()
@CommandHandler(CreateUserCommand)
export class CreateUserHandler implements ICommandHandler<CreateUserCommand> {
  constructor(
    private readonly userRepository: UserRepository,
    private readonly eventBus: EventBus,
  ) {}

  async execute(command: CreateUserCommand): Promise<User> {
    const { email, name, phone } = command;

    // Validate business rules
    const existingUser = await this.userRepository.findByEmail(email);
    if (existingUser) {
      throw new ConflictException('User already exists');
    }

    // Create user
    const user = await this.userRepository.create({
      email,
      name,
      phone,
    });

    // Publish domain event
    this.eventBus.publish(new UserCreatedEvent(user.id, user.email));

    return user;
  }
}
```

## Query Structure

### Query Definition
```typescript
import { IQuery } from '@nestjs/cqrs';

export class GetUserByIdQuery implements IQuery {
  constructor(public readonly userId: string) {}
}

export class GetUsersQuery implements IQuery {
  constructor(
    public readonly page: number = 1,
    public readonly limit: number = 10,
    public readonly search?: string,
  ) {}
}
```

### Query Handler
```typescript
import { QueryHandler, IQueryHandler } from '@nestjs/cqrs';
import { Injectable } from '@nestjs/common';

@Injectable()
@QueryHandler(GetUserByIdQuery)
export class GetUserByIdHandler implements IQueryHandler<GetUserByIdQuery> {
  constructor(private readonly userRepository: UserRepository) {}

  async execute(query: GetUserByIdQuery): Promise<User | null> {
    const { userId } = query;
    return this.userRepository.findById(userId);
  }
}

@Injectable()
@QueryHandler(GetUsersQuery)
export class GetUsersHandler implements IQueryHandler<GetUsersQuery> {
  constructor(private readonly userRepository: UserRepository) {}

  async execute(query: GetUsersQuery): Promise<PaginatedResult<User>> {
    const { page, limit, search } = query;
    return this.userRepository.findMany({
      page,
      limit,
      search,
    });
  }
}
```

## Event Structure

### Event Definition
```typescript
import { IEvent } from '@nestjs/cqrs';

export class UserCreatedEvent implements IEvent {
  constructor(
    public readonly userId: string,
    public readonly email: string,
    public readonly timestamp: Date = new Date(),
  ) {}
}
```

### Event Handler
```typescript
import { EventsHandler, IEventHandler } from '@nestjs/cqrs';
import { Injectable } from '@nestjs/common';

@Injectable()
@EventsHandler(UserCreatedEvent)
export class UserCreatedHandler implements IEventHandler<UserCreatedEvent> {
  constructor(
    private readonly emailService: EmailService,
    private readonly logger: Logger,
  ) {}

  async handle(event: UserCreatedEvent): Promise<void> {
    const { userId, email } = event;

    try {
      await this.emailService.sendWelcomeEmail(email);
      this.logger.log(`Welcome email sent to user ${userId}`);
    } catch (error) {
      this.logger.error(`Failed to send welcome email to ${email}`, error);
      // Don't throw - events should be resilient
    }
  }
}
```

## Controller Integration

### Using Commands and Queries in Controllers
```typescript
@Controller('users')
@ApiTags('Users')
export class UserController {
  constructor(
    private readonly commandBus: CommandBus,
    private readonly queryBus: QueryBus,
  ) {}

  @Post()
  @ApiOperation({ summary: 'Create a new user' })
  async createUser(@Body() dto: CreateUserDto): Promise<User> {
    const command = new CreateUserCommand(dto.email, dto.name, dto.phone);
    return this.commandBus.execute(command);
  }

  @Get(':id')
  @ApiOperation({ summary: 'Get user by ID' })
  async getUserById(@Param('id') id: string): Promise<User> {
    const query = new GetUserByIdQuery(id);
    const user = await this.queryBus.execute(query);
    
    if (!user) {
      throw new NotFoundException('User not found');
    }
    
    return user;
  }

  @Get()
  @ApiOperation({ summary: 'Get users with pagination' })
  async getUsers(@Query() dto: GetUsersDto): Promise<PaginatedResult<User>> {
    const query = new GetUsersQuery(dto.page, dto.limit, dto.search);
    return this.queryBus.execute(query);
  }
}
```

## Module Configuration

### CQRS Module Setup
```typescript
import { Module } from '@nestjs/common';
import { CqrsModule } from '@nestjs/cqrs';

// Import all handlers
import { CreateUserHandler } from './commands/handlers/create-user.handler';
import { GetUserByIdHandler } from './queries/handlers/get-user-by-id.handler';
import { GetUsersHandler } from './queries/handlers/get-users.handler';

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MiviaLabs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
