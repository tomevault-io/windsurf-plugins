---
trigger: always_on
description: Commands and Queries are the foundation of the CQRS (Command Query Responsibility Segregation) pattern in this application. They serve as data transfer objects (DTOs) that carry information between different layers of the application.
---


# Commands and Queries Instructions

## Overview
Commands and Queries are the foundation of the CQRS (Command Query Responsibility Segregation) pattern in this application. They serve as data transfer objects (DTOs) that carry information between different layers of the application.

**CRITICAL REQUIREMENTS**:
- Commands handle state-changing operations (POST, PUT, DELETE)
- Queries handle data retrieval operations (GET)
- All commands and queries must have a unique type identifier
- Use TypeScript interfaces for payload structure
- Follow consistent naming conventions

## Command Structure

### Basic Command Pattern
Commands represent intentions to change application state:

```typescript
import { Command } from "@tshio/command-bus";

export const CREATE_USER_COMMAND_TYPE = "users/CREATE_USER";

export interface CreateUserCommandPayload {
  firstName: string;
  lastName: string;
  email: string;
  role?: string;
}

export class CreateUserCommand implements Command<CreateUserCommandPayload> {
  public type: string = CREATE_USER_COMMAND_TYPE;

  constructor(public payload: CreateUserCommandPayload) {}
}
```

### Command Naming Conventions
- **Type constant**: `FEATURE_ACTION_COMMAND_TYPE` pattern
- **Interface**: `{Action}CommandPayload` 
- **Class**: `{Action}Command`
- **File**: `{action}.command.ts`

By {action}, we mean a verb that describes the operation, e.g., CreateUser, UpdateOrder, DeleteProduct.

Examples:
```typescript
// User management commands
export const CREATE_USER_COMMAND_TYPE = "users/CREATE_USER";
export const UPDATE_USER_COMMAND_TYPE = "users/UPDATE_USER";
export const DELETE_USER_COMMAND_TYPE = "users/DELETE_USER";
export const ACTIVATE_USER_COMMAND_TYPE = "users/ACTIVATE_USER";

// Order management commands  
export const CREATE_ORDER_COMMAND_TYPE = "orders/CREATE_ORDER";
export const PROCESS_ORDER_COMMAND_TYPE = "orders/PROCESS_ORDER";
export const CANCEL_ORDER_COMMAND_TYPE = "orders/CANCEL_ORDER";
```

### Command Payload Patterns

#### Create Commands
```typescript
// create-user.command.ts
export const CREATE_USER_COMMAND_TYPE = "users/CREATE_USER";

export interface CreateUserCommandPayload {
  firstName: string;
  lastName: string;
  email: string;
  role: UserRole;
  preferences?: UserPreferences;
}

export class CreateUserCommand implements Command<CreateUserCommandPayload> {
  public type: string = CREATE_USER_COMMAND_TYPE;

  constructor(public payload: CreateUserCommandPayload) {}
}
```

#### Update Commands
```typescript
// update-user.command.ts
export const UPDATE_USER_COMMAND_TYPE = "users/UPDATE_USER";

export interface UpdateUserCommandPayload {
  id: string;                    // Always required for updates
  firstName?: string;            // Optional fields for partial updates
  lastName?: string;
  email?: string;
  status?: UserStatus;
  updatedBy: string;             // Audit information
}

export class UpdateUserCommand implements Command<UpdateUserCommandPayload> {
  public type: string = UPDATE_USER_COMMAND_TYPE;

  constructor(public payload: UpdateUserCommandPayload) {}
}
```

#### Delete Commands
```typescript
// delete-user.command.ts
export const DELETE_USER_COMMAND_TYPE = "users/DELETE_USER";

export interface DeleteUserCommandPayload {
  id: string;
  deletedBy: string;             // Audit information
  reason?: string;               // Optional deletion reason
}

export class DeleteUserCommand implements Command<DeleteUserCommandPayload> {
  public type: string = DELETE_USER_COMMAND_TYPE;

  constructor(public payload: DeleteUserCommandPayload) {}
}
```

#### Business Logic Commands
```typescript
// activate-user.command.ts
export const ACTIVATE_USER_COMMAND_TYPE = "users/ACTIVATE_USER";

export interface ActivateUserCommandPayload {
  id: string;
  activatedBy: string;
  activationReason?: string;
}

export class ActivateUserCommand implements Command<ActivateUserCommandPayload> {
  public type: string = ACTIVATE_USER_COMMAND_TYPE;

  constructor(public payload: ActivateUserCommandPayload) {}
}
```

#### Complex Business Commands
```typescript
// process-order.command.ts
export const PROCESS_ORDER_COMMAND_TYPE = "orders/PROCESS_ORDER";

export interface ProcessOrderCommandPayload {
  orderId: string;
  items: OrderItem[];
  shippingAddress: Address;
  paymentMethod: PaymentMethod;
  couponCode?: string;
  specialInstructions?: string;
}

export interface OrderItem {
  productId: string;
  quantity: number;
  price: number;
  customization?: ProductCustomization;
}

export class ProcessOrderCommand implements Command<ProcessOrderCommandPayload> {
  public type: string = PROCESS_ORDER_COMMAND_TYPE;

  constructor(public payload: ProcessOrderCommandPayload) {}
}
```

## Query Structure

### Basic Query Pattern
Queries represent requests for data retrieval:

```typescript
import { Query } from "@tshio/query-bus";

export const USERS_QUERY_TYPE = "users/USERS";

export interface UsersQueryPayload {
  page?: number;
  limit?: number;
  sort?: Record<string, 'ASC' | 'DESC'>;
  filter?: UserFilters;
  search?: string;
}


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheSoftwareHouse/express-boilerplate](https://github.com/TheSoftwareHouse/express-boilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
