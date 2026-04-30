---
trigger: always_on
description: description: Apply repository pattern and consistent data access standards when implementing database operations to ensure maintainability, type safety, and proper error handling
---

___
description: Apply repository pattern and consistent data access standards when implementing database operations to ensure maintainability, type safety, and proper error handling
globs: "src/**/*.{ts,tsx}"
___

# Database Access Patterns

## Context

Database access should follow consistent patterns to ensure maintainable, secure, and efficient code. The repository pattern provides a standardized approach to data access that abstracts the underlying database implementation and enforces business rules consistently.

## Requirements

### Repository Pattern Implementation

- All database operations MUST be performed through repository classes, never directly with Prisma client in API routes or services
- Repositories MUST follow a consistent naming convention: `{Entity}Repository`
- Repository methods MUST include tenant isolation in all queries
- Repositories MUST use standardized error handling
- **Repositories SHOULD implement versioning** for audit-critical entities (see @081-data-versioning-standards.mdc)
- Repository methods MUST have consistent naming:
  - `findById`: Find a single entity by ID
  - `findByX`: Find entities by a specific attribute
  - `findAll`: Find all entities (with optional filtering)
  - `create`: Create a new entity
  - `update`: Update an existing entity
  - `delete`: Delete an entity
  - `count`: Count entities

### Multi-Tenant Data Access

- All database queries MUST include tenant context (`organizationId`)
- Cross-tenant operations MUST be explicitly authorized and audited
- Repository methods MUST validate tenant context for all operations

### Error Handling

- Repositories MUST catch all database errors and transform them to application-specific errors
- Database errors MUST NOT leak to API clients
- Error handling MUST be consistent across repositories
- Error messages MUST be user-friendly while preserving technical details for logging

### Transaction Management

- Complex operations affecting multiple entities MUST use transactions
- Transactions MUST be handled at the repository level
- Transaction boundaries MUST be clearly defined
- Transaction retry logic SHOULD be implemented for transient errors

### Query Performance

- Repositories SHOULD optimize database queries for performance
- N+1 query problems MUST be avoided by using proper eager loading
- Large result sets MUST be paginated

## Examples

<example>
// UserRepository with proper implementation
import { PrismaClient, User, Prisma } from '@prisma/client';
import { DatabaseError, NotFoundError } from '../errors/DatabaseError';

export class UserRepository {
  constructor(private prisma: PrismaClient) {}

  async findById(id: string, organizationId: string): Promise<User | null> {
    try {
      return await this.prisma.user.findFirst({
        where: {
          id,
          organizationId // Multi-tenancy enforcement
        }
      });
    } catch (error) {
      throw new DatabaseError('Failed to find user by ID', error);
    }
  }

  async findAll(
    organizationId: string,
    options?: {
      skip?: number;
      take?: number;
      orderBy?: Prisma.UserOrderByWithRelationInput;
    }
  ): Promise<User[]> {
    try {
      return await this.prisma.user.findMany({
        where: { organizationId },
        skip: options?.skip,
        take: options?.take,
        orderBy: options?.orderBy
      });
    } catch (error) {
      throw new DatabaseError('Failed to find users', error);
    }
  }

  async create(data: Prisma.UserCreateInput): Promise<User> {
    try {
      return await this.prisma.user.create({ data });
    } catch (error) {
      throw new DatabaseError('Failed to create user', error);
    }
  }

  async update(
    id: string,
    organizationId: string,
    data: Prisma.UserUpdateInput
  ): Promise<User> {
    try {
      return await this.prisma.user.update({
        where: {
          id,
          organizationId // Multi-tenancy enforcement
        },
        data
      });
    } catch (error) {
      throw new DatabaseError('Failed to update user', error);
    }
  }

  async createWithProfile(
    userData: Prisma.UserCreateInput,
    profileData: Prisma.ProfileCreateInput
  ): Promise<User> {
    try {
      // Use transaction to ensure atomic operation
      return await this.prisma.$transaction(async (tx) => {
        const user = await tx.user.create({
          data: userData
        });

        await tx.profile.create({
          data: {
            ...profileData,
            userId: user.id,
            organizationId: user.organizationId
          }
        });

        return user;
      });
    } catch (error) {
      throw new DatabaseError('Failed to create user with profile', error);
    }
  }
}
</example>

<example>
// API endpoint using repository pattern
import { NextApiRequest, NextApiResponse } from 'next';
import { UserRepository } from '../../../repositories/UserRepository';
import { withAuth } from '../../../middleware/auth';

export default withAuth(async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { organizationId } = req.auth;
  
  // Initialize repository

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spar65) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-15 -->
