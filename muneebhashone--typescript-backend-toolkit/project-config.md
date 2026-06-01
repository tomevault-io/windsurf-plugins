---
trigger: always_on
description: Mongoose model patterns for MongoDB schemas
---


# Mongoose Model Patterns

## Core Principle

Models define MongoDB schemas using Mongoose. Keep them simple and focused on data structure.

## Model Template

```typescript
import { Schema, model, type Document } from 'mongoose';

// TypeScript interface
export interface IModel extends Document {
  name: string;
  email: string;
  status: 'active' | 'inactive';
  metadata?: Record<string, any>;
  createdBy?: Schema.Types.ObjectId;
  createdAt: Date;
  updatedAt: Date;
}

// Mongoose schema
const schema = new Schema<IModel>(
  {
    name: {
      type: String,
      required: [true, 'Name is required'],
      trim: true,
      minlength: [2, 'Name must be at least 2 characters'],
      maxlength: [100, 'Name must not exceed 100 characters'],
    },
    email: {
      type: String,
      required: [true, 'Email is required'],
      unique: true,
      lowercase: true,
      trim: true,
      match: [/^\S+@\S+\.\S+$/, 'Please provide a valid email'],
    },
    status: {
      type: String,
      enum: {
        values: ['active', 'inactive'],
        message: 'Status must be either active or inactive',
      },
      default: 'active',
    },
    metadata: {
      type: Schema.Types.Mixed,
      default: {},
    },
    createdBy: {
      type: Schema.Types.ObjectId,
      ref: 'User',
    },
  },
  {
    timestamps: true, // Adds createdAt and updatedAt automatically
    collection: 'models', // Optional: specify collection name
  },
);

// Indexes for query performance
schema.index({ email: 1 }); // Single field index
schema.index({ status: 1, createdAt: -1 }); // Compound index
schema.index({ name: 'text' }); // Text index for search

// Virtual properties
schema.virtual('displayName').get(function () {
  return `${this.name} (${this.email})`;
});

// Instance methods
schema.methods.isActive = function () {
  return this.status === 'active';
};

schema.methods.toJSON = function () {
  const obj = this.toObject();
  delete obj.__v; // Remove version key
  return obj;
};

// Static methods
schema.statics.findActive = function () {
  return this.find({ status: 'active' });
};

schema.statics.findByEmail = function (email: string) {
  return this.findOne({ email: email.toLowerCase() });
};

// Pre-save hook
schema.pre('save', async function (next) {
  // Example: Normalize email
  if (this.isModified('email')) {
    this.email = this.email.toLowerCase().trim();
  }
  next();
});

// Post-save hook
schema.post('save', function (doc) {
  // Example: Log creation
  console.log('Document saved:', doc._id);
});

// Pre-remove hook
schema.pre('remove', async function (next) {
  // Example: Clean up related data
  await RelatedModel.deleteMany({ modelId: this._id });
  next();
});

// Create and export model
export const Model = model<IModel>('Model', schema);
```

## Common Field Types

### Basic Types

```typescript
{
  stringField: { type: String },
  numberField: { type: Number },
  booleanField: { type: Boolean },
  dateField: { type: Date },
  bufferField: { type: Buffer },
  mixedField: { type: Schema.Types.Mixed },
}
```

### References

```typescript
{
  userId: {
    type: Schema.Types.ObjectId,
    ref: "User", // Reference to User model
    required: true,
  },
}
```

### Arrays

```typescript
{
  tags: [String], // Array of strings
  items: [{ // Array of subdocuments
    name: String,
    quantity: Number,
  }],
  userIds: [{
    type: Schema.Types.ObjectId,
    ref: "User",
  }],
}
```

### Enums

```typescript
{
  status: {
    type: String,
    enum: {
      values: ["pending", "active", "inactive"],
      message: "Invalid status value",
    },
    default: "pending",
  },
}
```

### Nested Objects

```typescript
{
  address: {
    street: String,
    city: String,
    country: String,
    zipCode: String,
  },
}
```

## Field Options

### Common Options

```typescript
{
  field: {
    type: String,
    required: [true, "Error message"], // or just true
    unique: true, // Creates unique index
    index: true, // Creates index
    default: "value", // or function: () => Date.now()
    lowercase: true, // Auto-lowercase (String only)
    uppercase: true, // Auto-uppercase (String only)
    trim: true, // Remove whitespace (String only)
    minlength: 5, // Min length (String only)
    maxlength: 100, // Max length (String only)
    min: 0, // Min value (Number/Date only)
    max: 100, // Max value (Number/Date only)
    match: /regex/, // Regex validation (String only)
    validate: { // Custom validator
      validator: (v) => v > 0,
      message: "Must be positive",
    },
  },
}
```

## Indexes

Add indexes for frequently queried fields:

```typescript
// Single field index
schema.index({ email: 1 }); // 1 = ascending, -1 = descending

// Compound index
schema.index({ status: 1, createdAt: -1 });

// Text index for search
schema.index({ name: 'text', description: 'text' });

// Unique compound index
schema.index({ userId: 1, itemId: 1 }, { unique: true });

// Sparse index (only for documents with the field)
schema.index({ optionalField: 1 }, { sparse: true });

// TTL index (auto-delete after time)
schema.index({ expireAt: 1 }, { expireAfterSeconds: 0 });
```

## Hooks (Middleware)

### Pre hooks

```typescript
// Before save
schema.pre('save', async function (next) {
  // this = document being saved

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [muneebhashone/typescript-backend-toolkit](https://github.com/muneebhashone/typescript-backend-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
