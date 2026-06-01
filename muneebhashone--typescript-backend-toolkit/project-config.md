---
trigger: always_on
description: Service layer patterns for business logic and data access
---


# Service Layer Patterns

## Core Principle

Services contain business logic, database operations, external API calls, and complex computations. They should be framework-agnostic (no Express req/res).

## Service Template

```typescript
import { Model } from './module.model';
import { logger } from '@/plugins/logger';
import type { CreateInput, UpdateInput } from './module.dto';

/**
 * Find item by ID
 */
export const findById = async (id: string) => {
  const item = await Model.findById(id);
  return item;
};

/**
 * Find all items with pagination
 */
export const findAll = async (options: {
  page: number;
  limit: number;
  search?: string;
}) => {
  const { page, limit, search } = options;
  const skip = (page - 1) * limit;

  const query = search ? { name: { $regex: search, $options: 'i' } } : {};

  const [items, total] = await Promise.all([
    Model.find(query).skip(skip).limit(limit).lean(),
    Model.countDocuments(query),
  ]);

  return {
    data: items,
    pagination: {
      page,
      limit,
      total,
      totalPages: Math.ceil(total / limit),
    },
  };
};

/**
 * Create new item
 */
export const create = async (data: CreateInput) => {
  const item = await Model.create(data);

  logger.info('Item created', { itemId: item._id });

  return item.toObject();
};

/**
 * Update item
 */
export const update = async (
  id: string,
  data: UpdateInput,
  userId?: string,
) => {
  const item = await Model.findById(id);

  if (!item) {
    return null;
  }

  // Business logic: Check permissions
  if (item.createdBy?.toString() !== userId) {
    const error = new Error('Forbidden') as any;
    error.statusCode = 403;
    throw error;
  }

  Object.assign(item, data);
  await item.save();

  logger.info('Item updated', { itemId: id, userId });

  return item.toObject();
};

/**
 * Delete item
 */
export const remove = async (id: string, userId?: string) => {
  const item = await Model.findById(id);

  if (!item) {
    return false;
  }

  // Business logic: Check permissions
  if (item.createdBy?.toString() !== userId) {
    const error = new Error('Forbidden') as any;
    error.statusCode = 403;
    throw error;
  }

  await item.deleteOne();

  logger.info('Item deleted', { itemId: id, userId });

  return true;
};

/**
 * Complex business logic example
 */
export const performComplexOperation = async (input: {
  userId: string;
  data: any;
}) => {
  // 1. Validate business rules
  const user = await UserModel.findById(input.userId);
  if (!user) {
    throw new Error('User not found');
  }

  // 2. Perform operations
  const result = await Model.create({
    ...input.data,
    userId: input.userId,
  });

  // 3. Trigger background jobs if needed
  await triggerEmailJob(user.email, result);

  // 4. Return result
  return result;
};

/**
 * Trigger background job
 */
const triggerEmailJob = async (email: string, data: any) => {
  const { emailQueue } = await import('@/queues/email.queue');
  await emailQueue.add('sendNotification', { email, data });
};
```

## Key Patterns

### Database Operations

Use Mongoose models from `module.model.ts`:

```typescript
// Find
const item = await Model.findById(id);
const items = await Model.find({ status: 'active' });

// Create
const item = await Model.create({ name: 'Test' });

// Update
const item = await Model.findByIdAndUpdate(id, { name: 'New' }, { new: true });

// Delete
await Model.findByIdAndDelete(id);

// Count
const count = await Model.countDocuments({ status: 'active' });

// Use .lean() for better performance (returns plain objects)
const items = await Model.find().lean();
```

### Pagination Helper

Use pagination utility from [pagination.utils.ts](mdc:src/utils/pagination.utils.ts):

```typescript
import { getPaginator } from '@/utils/pagination.utils';

const paginatorInfo = getPaginator(limit, page, totalRecords);
const items = await Model.find()
  .skip(paginatorInfo.skip)
  .limit(paginatorInfo.limit);
```

Or implement manually:

```typescript
const skip = (page - 1) * limit;
const items = await Model.find().skip(skip).limit(limit);
const total = await Model.countDocuments();
```

### Background Jobs

Queue background tasks using BullMQ:

```typescript
import { emailQueue } from '@/queues/email.queue';

await emailQueue.add(
  'jobName',
  { data },
  {
    delay: 5000, // Optional: delay in ms
    attempts: 3, // Optional: retry attempts
  },
);
```

### Email Sending

Send emails through queue system:

```typescript
import { sendEmail } from '@/email/email.service';

await sendEmail({
  to: user.email,
  subject: 'Welcome',
  template: 'Welcome',
  data: { name: user.name },
});
```

### File Storage (S3)

Use storage service from [storage.ts](mdc:src/lib/storage.ts):

```typescript
import { uploadFile, deleteFile, getFileUrl } from '@/lib/storage';

// Upload file (usually in controller, after file is uploaded)
const { url, key } = await uploadFile({
  file: uploadedFile,
  key: `uploads/${userId}/${filename}`,
});

// Delete file
await deleteFile(fileKey);

// Get file URL
const url = getFileUrl(fileKey);
```

### Authentication & Tokens

Use auth utilities from the src/utils folder:

```typescript
import { signToken, verifyToken } from '@/utils/jwt.utils';

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [muneebhashone/typescript-backend-toolkit](https://github.com/muneebhashone/typescript-backend-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
